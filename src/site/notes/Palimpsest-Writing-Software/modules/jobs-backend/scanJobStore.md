---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/scan-job-store/","title":"scan_job.go — SQLite JobStore","tags":["file","jobs-backend"],"updated":"2026-03-05T09:17:03.773-07:00"}
---


# `scan_job.go`

**Module:** `[[modules/jobs-backend/overview|Jobs Backend]]`
**Path:** `backend/internal/repository/sqlite/scan_job.go`

> [!NOTE] SQLite implementation of the `[[modules/jobs-backend/types#JobStore|JobStore]]` interface using Ent ORM, providing CRUD operations for background job records with per-operation database connections.

## Purpose

This file implements the `JobStore` interface on the existing `sqlite.Store` struct, adding job persistence to the repository layer. It follows the same per-operation connection pattern used throughout the repository: each method calls `openEnt(ctx, dbPath)` to get a database client and defers `client.Close()`. Job records are stored in per-project SQLite database files at `storagePath/projectSlug.db`. The file also includes a compile-time interface check to guarantee that `Store` satisfies `JobStore`.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Store.CreateJob` | method | Creates a new job record with `pending` status |
| `Store.GetJob` | method | Retrieves a job by ID |
| `Store.GetActiveJob` | method | Finds a pending/running job for a given target |
| `Store.UpdateJobProgress` | method | Updates processed and matched counts |
| `Store.UpdateJobStatus` | method | Transitions job to a new status with timing fields |
| `scanJobToDTO` | function (unexported) | Converts `ent.ScanJob` to `jobs.JobDTO` |

---

## Functions

### `CreateJob(ctx context.Context, projectSlug string, in jobs.CreateJobInput) (*jobs.JobDTO, error)`

Inserts a new `ScanJob` record with `pending` status. Sets `job_type` from the input and optionally sets `target_id` if non-empty.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Identifies the project database file |
| `in` | `jobs.CreateJobInput` | yes | Contains `JobType` and `TargetID` |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | New job DTO | `*jobs.JobDTO` |
| DB open fails | Error | `error` |
| Insert fails | Error | `error` |

**Side Effects:** Writes a new row to the `scan_jobs` table.

---

### `GetJob(ctx context.Context, projectSlug, jobID string) (*jobs.JobDTO, error)`

Retrieves a single job by its UUID.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Identifies the project database file |
| `jobID` | `string` | yes | Job UUID |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | Job DTO | `*jobs.JobDTO` |
| Not found | Error (`"scan job not found: {id}"`) | `error` |
| DB error | Error | `error` |

> [!TIP] The "not found" error format (`"scan job not found: {id}"`) is checked by the `[[modules/jobs-backend/jobsHandler|HTTP handler]]` using `strings.Contains(err.Error(), "not found")` to return a 404 response.

---

### `GetActiveJob(ctx context.Context, projectSlug, targetID string) (*jobs.JobDTO, error)`

Finds a pending or running job for the given target. Queries with `target_id = targetID AND status IN (pending, running)`, ordered by `created_at DESC` (most recent first). Returns only the first match.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Identifies the project database file |
| `targetID` | `string` | yes | Module-specific target identifier |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Active job found | Job DTO | `*jobs.JobDTO` |
| No active job | `nil` (not an error) | `*jobs.JobDTO` |
| DB error | Error | `error` |

> [!IMPORTANT] Returns `nil, nil` when no active job exists. This is intentional -- callers should check for a nil result rather than checking for errors. This allows upstream code (e.g., glossary handler) to distinguish "no job running" from "database error" cleanly.

---

### `UpdateJobProgress(ctx context.Context, projectSlug, jobID string, processed, matched int) error`

Updates the `processed_items` and `matched_items` fields on an existing job record.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Identifies the project database file |
| `jobID` | `string` | yes | Job UUID |
| `processed` | `int` | yes | New processed item count |
| `matched` | `int` | yes | New matched item count |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | nil | `error` |
| DB error | Error | `error` |

**Side Effects:** Updates two columns in the `scan_jobs` table. The `updated_at` column is automatically set by Ent's `UpdateDefault`.

---

### `UpdateJobStatus(ctx context.Context, projectSlug, jobID, status string, errMsg *string) error`

Transitions the job to a new status and sets timing fields based on the transition:

| Status Transition | Timing Field Set |
| ----------------- | --------------- |
| Any -> `running` | `started_at = now` |
| Any -> `completed` | `completed_at = now` |
| Any -> `failed` | `completed_at = now`, `error_message = errMsg` |
| Any -> `cancelled` | `completed_at = now` |

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Identifies the project database file |
| `jobID` | `string` | yes | Job UUID |
| `status` | `string` | yes | New status value |
| `errMsg` | `*string` | no | Error message (only set when status is `failed`; may be nil) |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | nil | `error` |
| DB error | Error | `error` |

**Side Effects:** Updates status, timing, and optionally error message columns in the `scan_jobs` table.

---

### `scanJobToDTO(job *ent.ScanJob) *jobs.JobDTO` (unexported)

Converts an Ent `ScanJob` entity to a `[[modules/jobs-backend/types#JobDTO|jobs.JobDTO]]`. Performs direct field mapping with enum-to-string conversions for `JobType` and `Status`.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `job` | `*ent.ScanJob` | yes | Ent entity from database query |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Always | Mapped DTO | `*jobs.JobDTO` |

---

## Compile-Time Interface Check

```go
var _ jobs.JobStore = (*Store)(nil)
```

This line ensures at compile time that `sqlite.Store` fully implements the `[[modules/jobs-backend/types#JobStore|jobs.JobStore]]` interface. If any method is missing or has an incorrect signature, the build will fail.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `context` | stdlib | Context propagation |
| `fmt` | stdlib | Error formatting |
| `time` | stdlib | `time.Now()` for timing fields |
| `ent` | internal (generated) | Ent client and `IsNotFound` helper |
| `ent/scanjob` | internal (generated) | Field constants, enum types, and predicates |
| `jobs` | internal | `JobStore` interface, `JobDTO`, `CreateJobInput`, status constants |

## Side Effects

- **Database reads:** `GetJob`, `GetActiveJob` query the `scan_jobs` table
- **Database writes:** `CreateJob`, `UpdateJobProgress`, `UpdateJobStatus` modify the `scan_jobs` table

## Performance

Each method opens and closes its own Ent client connection via `openEnt(ctx, dbPath)` / `defer client.Close()`. This follows the same pattern used across all SQLite repository files in the codebase. While this incurs per-operation connection overhead, it avoids holding long-lived connections and is consistent with SQLite's single-writer concurrency model. The `[[modules/jobs-backend/manager|Manager]]` serializes job execution per project, so write contention is avoided at the application level.

## 12-Factor Compliance

The database path is derived from configuration (`storagePath`) rather than being hardcoded. The store receives its dependencies through the `Store` struct (storage path) and does not maintain any connection pool or in-memory state between operations.

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/types\|types.go — JobStore Interface]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobSchema\|ScanJob.go — Ent Schema]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/manager\|manager.go — Calls Store Methods]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/progress\|progress.go — Calls UpdateJobProgress and GetJob]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System Architecture]]
