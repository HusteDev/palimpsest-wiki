---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/overview/","title":"Jobs Backend Module","tags":["module","jobs-backend"],"updated":"2026-03-05T09:11:42.309-07:00"}
---


# Jobs Backend

> [!NOTE] Go package providing a generic background job engine with per-project worker goroutines, handler registration and dispatch, cooperative cancellation via database polling, progress reporting, and graceful shutdown coordination. Module-specific handlers (`TermScanHandler`, `ReindexHandler`) implement the `JobHandler` interface and are registered at startup.

## Responsibilities

- Provide the `JobHandler` interface for module-specific job implementations
- Provide the `JobStore` interface for database persistence (implemented by SQLite repository)
- Provide the `ProgressReporter` interface for progress updates and cancellation checks
- Manage per-project worker goroutines with buffered job channels (capacity: 100)
- Dispatch submitted jobs to registered handlers by job type string
- Persist job lifecycle state transitions (`pending` → `running` → `completed`/`failed`/`cancelled`)
- Support cooperative cancellation: handlers poll `IsCancelled()` between processing steps
- Coordinate graceful shutdown: wait for in-flight jobs with context-based timeout
- Serve HTTP endpoints for job status queries and cancellation requests

## Public API Surface

### Manager

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `NewManager(store)` | constructor | Create Manager with a `JobStore` for persistence |
| `RegisterHandler(handler)` | method | Register a `JobHandler` for its job type (panics on duplicate) |
| `Submit(ctx, projectSlug, jobType, targetID, params)` | method | Create job record, queue for execution, return job ID |
| `GetJobStatus(ctx, projectSlug, jobID)` | method | Return current `*JobDTO` for a job |
| `CancelJob(ctx, projectSlug, jobID)` | method | Transition job to `"cancelled"` (only pending/running) |
| `Shutdown(ctx)` | method | Close shutdown channel, wait for workers with context timeout |

### Interfaces

| Interface | Description |
| --------- | ----------- |
| `JobHandler` | Module contract: `JobType() string`, `Execute(ctx, JobContext) error` |
| `ProgressReporter` | Progress contract: `Update(processed, matched) error`, `IsCancelled() bool` |
| `JobStore` | Persistence contract: `CreateJob`, `GetJob`, `GetActiveJob`, `UpdateJobProgress`, `UpdateJobStatus` |

### Types

| Type | Source | Description |
| ---- | ------ | ----------- |
| `JobDTO` | types.go | Job record with all fields (ID, status, progress, timestamps) |
| `CreateJobInput` | types.go | Input for creating a new job (JobType, TargetID) |
| `JobContext` | types.go | Context passed to handlers (JobID, ProjectSlug, TargetID, Params, Progress) |

### Status Constants

| Constant | Value | Description |
| -------- | ----- | ----------- |
| `StatusPending` | `"pending"` | Job created, waiting in queue |
| `StatusRunning` | `"running"` | Worker executing handler |
| `StatusCompleted` | `"completed"` | Handler succeeded |
| `StatusFailed` | `"failed"` | Handler returned error |
| `StatusCancelled` | `"cancelled"` | User requested cancellation |

### HTTP Handler (handlers.JobHandler)

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `NewJobHandler(manager)` | constructor | Create handler (nil manager → 503 on all endpoints) |
| `Jobs(w, r)` | method | Router: extract slug + job ID, dispatch GET/DELETE |

## Internal Structure

**Job Engine** — Core types and worker management:
- `[[modules/jobs-backend/types|types.go]]` — `JobHandler`, `ProgressReporter`, `JobStore` interfaces; `JobDTO`, `CreateJobInput`, `JobContext` types; status constants
- `[[modules/jobs-backend/manager|manager.go]]` — `Manager` struct, `NewManager`, handler registration, job submission, per-project workers, job processing, graceful shutdown
- `[[modules/jobs-backend/progress|progress.go]]` — `progressReporter` implementation delegating to `JobStore`

**HTTP Handler** — API endpoints:
- `[[modules/jobs-backend/jobsHandler|jobs.go]]` — `JobHandler` struct, `Jobs` router, `getJob`/`cancelJob` handlers, `JobResponse` DTO, `extractJobID` URL parser

**Database** — Ent schema and SQLite implementation:
- `[[modules/jobs-backend/scanJobSchema|ScanJob.go]]` — Ent schema with job_type enum, status enum, progress fields, timing fields, 3 indexes
- `[[modules/jobs-backend/scanJobStore|scan_job.go]]` — SQLite `JobStore` implementation via Ent queries

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `context` | external (stdlib) | Context propagation for cancellation |
| `sync` | external (stdlib) | `RWMutex` for worker map, `WaitGroup` for shutdown |
| `fmt` | external (stdlib) | Error formatting |
| `time` | external (stdlib) | Timestamps on DTOs |
| `logging` | internal | `WithComponent("jobs")` for structured logging |
| `contextx` | internal | `GetProjectSlugFromRequest` for HTTP routing |
| `response` | internal | HTTP response helpers |
| `ent` | internal (generated) | `ScanJob` entity, `scanjob` predicates |

## Integration Points

- **Glossary module** — `glossary.TermScanHandler` registered at startup; glossary HTTP handler calls `Submit()` for `"term_scan"` jobs when entries are created, updated, or deleted
- **Search module** — `search.ReindexHandler` registered at startup; search HTTP handler calls `Submit()` for `"search_reindex"` jobs (content/glossary loaders currently stubbed)
- **HTTP server** — `Server` receives `*jobs.Manager` as optional constructor parameter; routes `/api/projects/{slug}/jobs/*` to `JobHandler`
- **Repository** — `sqlite.Store` implements `jobs.JobStore`; job records stored in per-project `.db` files
- **Startup** — `main.go` creates `Manager`, registers all handlers, passes to `NewServer()`

## Logging

- `Manager`: `logging.GetGlobal().WithComponent("jobs")`
- `JobHandler` (HTTP): `logging.GetGlobal().WithComponent("handler-jobs")`
- `TermScanHandler`: `logging.GetGlobal().WithComponent("term-scan")`
- `ReindexHandler`: `logging.GetGlobal().WithComponent("search-reindex")`

## Related

- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System]]
- [[Palimpsest-Writing-Software/api/jobs/overview\|Jobs API Service]]
- [[Palimpsest-Writing-Software/features/glossary/overview\|Glossary System]]
- [[Palimpsest-Writing-Software/features/search/overview\|Search Engine]]
