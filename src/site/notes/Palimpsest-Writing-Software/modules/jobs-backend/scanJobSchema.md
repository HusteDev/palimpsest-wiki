---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/scan-job-schema/","title":"ScanJob.go — Ent Schema","tags":["file","jobs-backend"],"updated":"2026-03-05T09:16:14.572-07:00"}
---


# `ScanJob.go`

**Module:** `[[modules/jobs-backend/overview|Jobs Backend]]`
**Path:** `backend/internal/ent/schema/ScanJob.go`

> [!NOTE] Ent ORM schema defining the `ScanJob` database table for background job tracking, including job type and status enums, progress counters, error handling, timing fields, and composite indexes.

## Purpose

This file defines the `ScanJob` Ent schema, which creates the database table used to persist background job records. The schema tracks the full lifecycle of a job from creation (`pending`) through execution (`running`) to a terminal state (`completed`, `failed`, or `cancelled`). It uses Ent enums for both `job_type` and `status` to enforce valid values at the database level. The `target_id` field is intentionally a plain string (not a foreign key edge) to allow cross-module flexibility -- different job types reference different entity types.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ScanJob` | Ent schema | Schema struct implementing `ent.Schema` |
| `ScanJob.Fields()` | method | Returns field definitions for the table |
| `ScanJob.Edges()` | method | Returns nil (no edges by design) |
| `ScanJob.Indexes()` | method | Returns 3 composite/single indexes |

---

## Fields

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `id` | `string` (UUID) | yes | `uuid.New().String()` | Primary key, auto-generated UUID |
| `job_type` | `enum` | yes | `"term_scan"` | Type of background job; maps to a registered `[[modules/jobs-backend/types#JobHandler|JobHandler]]` |
| `status` | `enum` | yes | `"pending"` | Current lifecycle status |
| `target_id` | `string` | no (optional) | — | Module-specific target identifier (e.g., glossary term UUID) |
| `total_items` | `int` | yes | `0` | Total number of items to process |
| `processed_items` | `int` | yes | `0` | Number of items processed so far |
| `matched_items` | `int` | yes | `0` | Number of items with matches found |
| `error_message` | `string` | no (optional) | — | Error details if job failed |
| `started_at` | `time.Time` | no (optional, nillable) | — | When the job started processing |
| `completed_at` | `time.Time` | no (optional, nillable) | — | When the job finished (any terminal state) |
| `created_at` | `time.Time` | yes | `time.Now` | Record creation timestamp |
| `updated_at` | `time.Time` | yes | `time.Now` (auto-updated) | Last modification timestamp |

### `job_type` Enum Values

| Value | Description |
| ----- | ----------- |
| `term_scan` | Scan content in scope for a specific glossary term |
| `full_rescan` | Re-scan entire project (manual trigger, future use) |
| `search_reindex` | Rebuild all FTS5 search indexes for a project |

### `status` Enum Values

| Value | Description |
| ----- | ----------- |
| `pending` | Job created, waiting in queue |
| `running` | Worker executing handler |
| `completed` | Handler succeeded |
| `failed` | Handler returned error |
| `cancelled` | User requested cancellation via API |

---

## Edges

None. This is intentional.

> [!IMPORTANT] The `ScanJob` schema has no Ent edges. The `target_id` field is a plain string rather than a foreign key because it references different entity types depending on the `job_type`. For example, a `term_scan` job targets a glossary entry UUID, while a `search_reindex` job may target a project slug. This design provides cross-module flexibility without coupling the job table to any specific entity schema.

---

## Indexes

| Fields | Purpose |
| ------ | ------- |
| `(target_id, status)` | Find active jobs for a specific target (e.g., check if a pending/running job already exists for a glossary term) |
| `(status)` | Find all jobs by status for monitoring and cleanup |
| `(created_at)` | Find recent jobs for listing and pagination |

---

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `time` | stdlib | Default timestamp values |
| `entgo.io/ent` | external | Ent schema framework |
| `entgo.io/ent/schema/field` | external | Field type definitions |
| `entgo.io/ent/schema/index` | external | Index definitions |
| `github.com/google/uuid` | external | UUID generation for primary key |

## Side Effects

None. This is a schema definition file. The actual table is created during Ent code generation and database migration.

## Performance

- The `(target_id, status)` composite index is the most frequently queried index, used by `[[modules/jobs-backend/scanJobStore#GetActiveJob|GetActiveJob()]]` to check for duplicate active jobs before submitting new ones.
- The `(status)` index supports monitoring queries that filter by job status across all targets.
- The `(created_at)` index supports ordered listing for future job history endpoints.

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/types\|types.go — JobDTO Maps to This Schema]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobStore\|scan_job.go — SQLite Implementation]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/manager\|manager.go — Creates Records via Store]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System Architecture]]
