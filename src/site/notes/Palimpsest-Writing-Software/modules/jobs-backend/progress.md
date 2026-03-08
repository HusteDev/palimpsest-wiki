---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/progress/","title":"progress.go — Progress Reporter","tags":["file","jobs-backend"],"updated":"2026-03-05T09:14:55.671-07:00"}
---


# `progress.go`

**Module:** `[[modules/jobs-backend/overview|Jobs Backend]]`
**Path:** `backend/internal/jobs/progress.go`

> [!NOTE] Concrete implementation of the `[[modules/jobs-backend/types#ProgressReporter|ProgressReporter]]` interface that delegates progress updates and cancellation checks to the `[[modules/jobs-backend/types#JobStore|JobStore]]`.

## Purpose

This file provides the `progressReporter` struct, an internal implementation created by `[[modules/jobs-backend/manager|Manager.processJob()]]` for each job execution. It wraps the `JobStore` to persist progress counts and polls the database to detect cooperative cancellation. Handlers receive a `progressReporter` via the `[[modules/jobs-backend/types#JobContext|JobContext.Progress]]` field and use it to report progress and check whether the user has cancelled the job via the API.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `progressReporter` | struct (unexported) | Implements `ProgressReporter` by delegating to `JobStore` |
| `newProgressReporter` | constructor (unexported) | Creates a `progressReporter` for a specific job |

---

## `progressReporter` Struct

| Field | Type | Access | Description |
| ----- | ---- | ------ | ----------- |
| `store` | `JobStore` | private | Persistence layer for progress updates and status checks |
| `ctx` | `context.Context` | private | Context for database operations |
| `projectSlug` | `string` | private | Project identifier for store calls |
| `jobID` | `string` | private | Job UUID for store calls |
| `log` | `*logging.Logger` | private | Logger inherited from Manager (component: `"jobs"`) |

---

## Functions

### `newProgressReporter(store JobStore, ctx context.Context, projectSlug, jobID string, log *logging.Logger) *progressReporter`

Creates a new progress reporter for a specific job. Called internally by `[[modules/jobs-backend/manager#processJob|Manager.processJob()]]`.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `store` | `JobStore` | yes | Persistence layer for updates |
| `ctx` | `context.Context` | yes | Context for database operations |
| `projectSlug` | `string` | yes | Project identifier |
| `jobID` | `string` | yes | Job UUID |
| `log` | `*logging.Logger` | yes | Logger instance |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Always | Initialized reporter | `*progressReporter` |

---

### `Update(processed, matched int) error`

Persists the current progress counts (processed items and matched items) to the database by delegating to `store.UpdateJobProgress()`. Logs a warning if the database update fails but still returns the error to the caller.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `processed` | `int` | yes | Number of items processed so far |
| `matched` | `int` | yes | Number of items where matches were found |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | nil | `error` |
| DB update fails | Error (also logged as warning) | `error` |

> [!TIP] Handlers should call `Update()` periodically (e.g., every 10 items) rather than after every single item to reduce database write pressure.

---

### `IsCancelled() bool`

Checks whether the job has been cancelled by querying the current job status from the database. Returns `true` if the status is `"cancelled"`, `false` otherwise.

| Condition | Returns | Description |
| --------- | ------- | ----------- |
| Status is `cancelled` | `true` | Handler should return early |
| Status is any other value | `false` | Handler should continue |
| Database error | `false` | Safe default: do not interrupt work on transient errors |

> [!IMPORTANT] If the status check fails due to a database error, `IsCancelled()` returns `false` as a safe default. This design choice avoids interrupting a running job due to a transient database issue. The error is logged as a warning.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `context` | stdlib | Context propagation for database operations |
| `logging` | internal | Structured logging for warnings |

## Side Effects

- **Database reads:** `IsCancelled()` queries the job record on every call
- **Database writes:** `Update()` writes processed/matched counts to the job record

## Notes

> [!WARNING] `IsCancelled()` performs a full database query (`store.GetJob()`) on every call. Handlers that check cancellation frequently should balance responsiveness against database load. Checking every 10-50 items is a reasonable interval.

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/types\|types.go — ProgressReporter Interface]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/manager\|manager.go — processJob Creates Reporter]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobStore\|scan_job.go — JobStore Implementation]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System Architecture]]
