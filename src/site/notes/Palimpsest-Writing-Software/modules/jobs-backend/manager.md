---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/manager/","title":"manager.go — Job Manager","tags":["file","jobs-backend"],"updated":"2026-03-05T09:14:25.676-07:00"}
---


# `manager.go`

**Module:** `[[modules/jobs-backend/overview|Jobs Backend]]`
**Path:** `backend/internal/jobs/manager.go`

> [!NOTE] Core job engine implementation providing handler registration, job submission, per-project worker goroutines, job dispatch, and graceful shutdown coordination.

## Purpose

The Manager is the central orchestrator of the job system. It maintains a registry of `[[modules/jobs-backend/types#JobHandler|JobHandler]]` implementations, creates database records for submitted jobs, ensures one worker goroutine exists per project (to avoid SQLite write contention), and dispatches queued jobs to the correct handler. It also manages the full job lifecycle state machine (`pending` -> `running` -> `completed`/`failed`) and coordinates graceful shutdown by waiting for all in-flight jobs to finish.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Manager` | struct | Central job engine that manages handlers, workers, and job dispatch |
| `NewManager` | constructor | Creates a new Manager with a `[[modules/jobs-backend/types#JobStore|JobStore]]` |
| `Manager.RegisterHandler` | method | Registers a `JobHandler` for its job type |
| `Manager.Submit` | method | Creates a job record and queues it for execution |
| `Manager.GetJobStatus` | method | Returns the current `*JobDTO` for a job |
| `Manager.CancelJob` | method | Transitions a pending/running job to `cancelled` |
| `Manager.Shutdown` | method | Stops all workers gracefully with context timeout |

---

## Manager Struct

| Field | Type | Access | Description |
| ----- | ---- | ------ | ----------- |
| `store` | `JobStore` | private | Persistence layer for job records |
| `handlers` | `map[string]JobHandler` | private | Registry of handlers keyed by job type string |
| `log` | `*logging.Logger` | private | Logger with component `"jobs"` |
| `mu` | `sync.RWMutex` | private | Guards `projectJobs` map |
| `projectJobs` | `map[string]chan jobRequest` | private | Per-project buffered channels (capacity 100) |
| `shutdown` | `chan struct{}` | private | Closed to signal all workers to stop |
| `wg` | `sync.WaitGroup` | private | Tracks active worker goroutines for shutdown |

### Internal Type: `jobRequest`

Internal message struct sent to a project's worker goroutine via the buffered channel.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `JobID` | `string` | UUID of the job record in the database |
| `ProjectSlug` | `string` | Project this job belongs to |
| `JobType` | `string` | Must match a registered handler's `JobType()` |
| `TargetID` | `string` | Module-specific target identifier |
| `Params` | `map[string]any` | Optional module-specific parameters |

---

## Functions

### `NewManager(store JobStore) *Manager`

Creates a new Manager with the given store. Initializes the handler registry, project jobs map, and shutdown channel. Logger is configured via `logging.GetGlobal().WithComponent("jobs")`.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `store` | `JobStore` | yes | Persistence layer for job records |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Always | Initialized Manager | `*Manager` |

---

### `RegisterHandler(handler JobHandler)`

Registers a `JobHandler` for its job type string. Must be called during server startup before any jobs are submitted.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `handler` | `JobHandler` | yes | Handler implementing `JobType()` and `Execute()` |

> [!WARNING] Panics if a handler is already registered for the same job type. This is intentional to catch configuration errors at startup rather than silently dropping jobs at runtime.

---

### `Submit(ctx context.Context, projectSlug, jobType, targetID string, params map[string]any) (string, error)`

Creates a job record in the database and queues it for execution. Returns the job ID for status polling.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Project identifier |
| `jobType` | `string` | yes | Must match a registered handler |
| `targetID` | `string` | yes | Module-specific target (e.g., term ID) |
| `params` | `map[string]any` | no | Optional parameters passed to handler (may be nil) |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | Job UUID | `string` |
| No handler registered | Error | `error` |
| DB create fails | Error | `error` |

**Behavior:**
1. Validates that a handler exists for the given `jobType`
2. Creates a job record in the database via `store.CreateJob()` with `pending` status
3. Calls `ensureWorker()` to guarantee a worker goroutine exists for this project
4. Queues a `jobRequest` on the project's buffered channel (non-blocking)
5. If the channel buffer is full, the job remains in `pending` state in the database and will be picked up when the worker drains the queue

---

### `GetJobStatus(ctx context.Context, projectSlug, jobID string) (*JobDTO, error)`

Returns the current status of a job by delegating to `store.GetJob()`.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Project identifier |
| `jobID` | `string` | yes | Job UUID |

---

### `CancelJob(ctx context.Context, projectSlug, jobID string) error`

Attempts to cancel a pending or running job by transitioning its status to `cancelled`.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Request context |
| `projectSlug` | `string` | yes | Project identifier |
| `jobID` | `string` | yes | Job UUID |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Success | nil | `error` |
| Job not found | Error | `error` |
| Job already terminal | Error (`"cannot cancel job in status..."`) | `error` |

> [!IMPORTANT] Cancellation is cooperative. Setting the status to `cancelled` in the database does not immediately stop the handler. The handler must call `[[modules/jobs-backend/progress|progressReporter.IsCancelled()]]` between processing steps and return early if true.

---

### `Shutdown(ctx context.Context) error`

Stops all workers gracefully. Closes the `shutdown` channel to signal all worker goroutines, then waits for the `WaitGroup` with a context-based timeout.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `ctx` | `context.Context` | yes | Timeout context for shutdown |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| All workers stopped | nil | `error` |
| Context deadline exceeded | `ctx.Err()` | `error` |

---

## Internal Methods

### `ensureWorker(projectSlug string)`

Starts a worker goroutine for a project if one does not already exist. Creates a buffered channel with capacity 100 and stores it in `projectJobs` under a write lock. Increments the `WaitGroup` before launching the goroutine.

### `getJobChannel(projectSlug string) chan jobRequest`

Returns the buffered job channel for a project under a read lock. Called by `Submit()` to queue job requests.

### `runWorker(projectSlug string, jobs <-chan jobRequest)`

Select loop that runs as a goroutine for a single project. Listens on two channels:
- `shutdown` — returns immediately, stopping the worker
- `jobs` — receives the next `jobRequest` and calls `processJob()`

Calls `wg.Done()` on exit via `defer`.

### `processJob(req jobRequest)`

Executes a single job by dispatching to the registered handler:
1. Looks up handler by `req.JobType`; if missing, marks job as `failed`
2. Transitions status from `pending` to `running` via `store.UpdateJobStatus()`
3. Creates a `[[modules/jobs-backend/progress|progressReporter]]` for the job
4. Builds a `[[modules/jobs-backend/types#JobContext|JobContext]]` with the progress reporter
5. Calls `handler.Execute(ctx, jobCtx)`
6. On error: marks job as `failed` with the error message
7. On success: checks `IsCancelled()` (handler may have been cancelled during execution); if not cancelled, marks as `completed`

> [!TIP] The `processJob` method uses `context.Background()` rather than a request context because jobs run asynchronously after the HTTP request that submitted them has already returned.

---

## Worker Strategy

> [!IMPORTANT] The Manager uses **one goroutine per project** to serialize job execution within a project. This avoids SQLite write contention since each project has its own `.db` file with a single-writer lock. Jobs across different projects execute concurrently.

## Performance

- **Channel buffer size:** 100 per project. If the buffer fills, `Submit()` logs a warning but the job record still exists in `pending` state in the database.
- **Worker lifecycle:** Workers are created on-demand when a project first submits a job and run until shutdown. They are never cleaned up during normal operation.
- **Shutdown:** Uses `sync.WaitGroup` with context timeout to bound the maximum shutdown wait time.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `context` | stdlib | Context propagation |
| `fmt` | stdlib | Error formatting |
| `sync` | stdlib | `RWMutex` for worker map, `WaitGroup` for shutdown |
| `logging` | internal | `WithComponent("jobs")` for structured logging |

## Side Effects

- **Goroutine creation:** `ensureWorker()` spawns a long-lived goroutine per project
- **Channel creation:** Buffered channels (capacity 100) created per project in `projectJobs` map
- **Database writes:** `processJob()` transitions job status through the store

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/types\|types.go — Type Definitions]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/progress\|progress.go — Progress Reporter]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/jobsHandler\|jobs.go — HTTP Handler]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobStore\|scan_job.go — SQLite JobStore]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System Architecture]]
