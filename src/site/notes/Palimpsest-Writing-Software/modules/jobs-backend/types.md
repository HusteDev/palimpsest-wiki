---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/types/","title":"types.go — Job Engine Type Definitions","tags":["file","jobs-backend"],"updated":"2026-03-05T09:13:30.842-07:00"}
---


# `types.go`

**Module:** `[[modules/jobs-backend/overview|Jobs Backend]]`
**Path:** `backend/internal/jobs/types.go`

> [!NOTE] Defines all module-agnostic interfaces, DTOs, and constants for the generic job engine. Modules such as glossary and search provide their own `JobHandler` implementations to plug into the engine.

## Purpose

This file establishes the contract layer for the entire jobs subsystem. It defines the three core interfaces (`JobHandler`, `ProgressReporter`, `JobStore`) that decouple the job engine from specific modules and from the persistence layer. It also defines the data transfer objects (`JobDTO`, `CreateJobInput`, `JobContext`) used to pass job data between layers, and the status constants that govern the job lifecycle state machine.

## Exports

### Interfaces

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `JobHandler` | interface | Contract for module-specific job execution; registered with `[[modules/jobs-backend/manager|Manager]]` at startup |
| `ProgressReporter` | interface | Contract for progress updates and cooperative cancellation checking |
| `JobStore` | interface | Persistence contract for job CRUD; implemented by `[[modules/jobs-backend/scanJobStore|sqlite.Store]]` |

### Types

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `JobDTO` | struct | Full job record returned by the store layer |
| `CreateJobInput` | struct | Input fields for creating a new job record |
| `JobContext` | struct | Execution context passed to `JobHandler.Execute()` |

### Constants

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `StatusPending` | string | `"pending"` — job created, waiting in queue |
| `StatusRunning` | string | `"running"` — worker executing handler |
| `StatusCompleted` | string | `"completed"` — handler succeeded |
| `StatusFailed` | string | `"failed"` — handler returned error |
| `StatusCancelled` | string | `"cancelled"` — user requested cancellation |

---

## Interface Details

### `JobHandler`

Implemented by each module to handle its specific job type. Handlers are registered with `[[modules/jobs-backend/manager|Manager.RegisterHandler()]]` during server startup.

| Method | Signature | Returns | Description |
| ------ | --------- | ------- | ----------- |
| `JobType` | `JobType() string` | `string` | Returns the job type string this handler processes (e.g., `"term_scan"`) |
| `Execute` | `Execute(ctx context.Context, job JobContext) error` | `error` | Runs the job; should call `progress.Update()` periodically and check `progress.IsCancelled()` |

> [!TIP] Handlers should return `nil` on success and a non-nil `error` on failure. The `[[modules/jobs-backend/manager|Manager]]` translates the return value into the appropriate status transition (`completed` or `failed`).

### `ProgressReporter`

Allows handlers to report progress and check for cooperative cancellation. The concrete implementation is `[[modules/jobs-backend/progress|progressReporter]]`.

| Method | Signature | Returns | Description |
| ------ | --------- | ------- | ----------- |
| `Update` | `Update(processed, matched int) error` | `error` | Persists current progress counts to the database |
| `IsCancelled` | `IsCancelled() bool` | `bool` | Checks whether the job has been cancelled via the API |

### `JobStore`

Defines persistence operations for job records. Decouples the engine from Ent and the full `ProjectStore` interface. Implemented by `[[modules/jobs-backend/scanJobStore|sqlite.Store]]`.

| Method | Signature | Returns | Description |
| ------ | --------- | ------- | ----------- |
| `CreateJob` | `CreateJob(ctx, projectSlug string, in CreateJobInput) (*JobDTO, error)` | `*JobDTO, error` | Inserts a new job record with `"pending"` status |
| `GetJob` | `GetJob(ctx, projectSlug, jobID string) (*JobDTO, error)` | `*JobDTO, error` | Retrieves a job by ID |
| `GetActiveJob` | `GetActiveJob(ctx, projectSlug, targetID string) (*JobDTO, error)` | `*JobDTO, error` | Finds a pending or running job for the given target; returns `nil` (not error) if none |
| `UpdateJobProgress` | `UpdateJobProgress(ctx, projectSlug, jobID string, processed, matched int) error` | `error` | Updates the processed and matched counts |
| `UpdateJobStatus` | `UpdateJobStatus(ctx, projectSlug, jobID, status string, errMsg *string) error` | `error` | Transitions job to new status; sets `started_at`/`completed_at` as appropriate |

---

## Type Details

### `JobDTO`

Represents a job record as returned by the store. All timestamps use `time.Time`.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `ID` | `string` | UUID primary key |
| `JobType` | `string` | Job type string (e.g., `"term_scan"`, `"search_reindex"`) |
| `Status` | `string` | Current status (`pending`, `running`, `completed`, `failed`, `cancelled`) |
| `TargetID` | `string` | Module-specific target identifier (e.g., glossary term UUID) |
| `TotalItems` | `int` | Total number of items to process |
| `ProcessedItems` | `int` | Number of items processed so far |
| `MatchedItems` | `int` | Number of items where matches were found |
| `ErrorMessage` | `string` | Error details if job failed; empty otherwise |
| `StartedAt` | `*time.Time` | When the job started processing (nil if still pending) |
| `CompletedAt` | `*time.Time` | When the job finished (nil if not yet terminal) |
| `CreatedAt` | `time.Time` | Record creation timestamp |
| `UpdatedAt` | `time.Time` | Last modification timestamp |

### `CreateJobInput`

Contains the minimum fields needed to create a new job record.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `JobType` | `string` | Must match a registered handler's `JobType()` return value |
| `TargetID` | `string` | Module-specific target (e.g., term ID, entity ID) |

### `JobContext`

Passed to `JobHandler.Execute()` with all the information a handler needs to run.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `JobID` | `string` | Unique identifier for this job |
| `ProjectSlug` | `string` | Identifies the project this job belongs to |
| `TargetID` | `string` | Module-specific target (e.g., term ID, entity ID) |
| `Params` | `map[string]any` | Optional module-specific parameters (e.g., `{"isUpdate": true}`) |
| `Progress` | `ProgressReporter` | Interface for progress reporting and cancellation checking |

---

## Status Constants

These constants define the valid states in the job lifecycle state machine:

```
pending → running → completed
                  → failed
          → cancelled (from pending or running)
```

| Constant | Value | Description |
| -------- | ----- | ----------- |
| `StatusPending` | `"pending"` | Job created, waiting in queue |
| `StatusRunning` | `"running"` | Worker goroutine executing handler |
| `StatusCompleted` | `"completed"` | Handler returned nil (success) |
| `StatusFailed` | `"failed"` | Handler returned non-nil error |
| `StatusCancelled` | `"cancelled"` | Cancelled via API; handler checks `IsCancelled()` |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `context` | stdlib | Context propagation for cancellation |
| `time` | stdlib | Timestamp types on `JobDTO` |

## Side Effects

None. This file contains only type definitions and constants with no module-level side effects.

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/manager\|manager.go — Job Manager]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/progress\|progress.go — Progress Reporter]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobStore\|scan_job.go — SQLite JobStore]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobSchema\|ScanJob.go — Ent Schema]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System Architecture]]
