---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/architecture/jobs/overview/","title":"Jobs System","tags":["architecture","jobs","background-processing","infrastructure"],"updated":"2026-03-05T09:11:08.146-07:00"}
---


# Jobs System

> [!NOTE] Generic background job engine with per-project worker goroutines, handler-based dispatch, progress reporting, cooperative cancellation, and graceful shutdown. Modules register `JobHandler` implementations at startup; the Manager routes submitted jobs to the correct handler and persists lifecycle state to the database.

## Overview

The Palimpsest jobs system provides a generic framework for running asynchronous background tasks. Rather than each module implementing its own goroutine management, all background work flows through a centralized `Manager` that handles worker lifecycle, database persistence, progress tracking, and cancellation.

The architecture follows a **handler registration pattern**: each module (glossary, search, etc.) implements the `JobHandler` interface and registers it during server startup. When a job is submitted, the Manager looks up the correct handler by job type string and dispatches execution to a per-project worker goroutine.

| Concept | Description |
| ------- | ----------- |
| **Manager** | Central coordinator: registers handlers, submits jobs, manages workers, handles shutdown |
| **JobHandler** | Module-specific interface: declares a `JobType()` string and an `Execute()` method |
| **JobStore** | Repository interface: persists job records to the database via Ent ORM |
| **ProgressReporter** | Callback interface: lets handlers report progress and check for cancellation |
| **Worker** | Per-project goroutine: processes jobs sequentially from a buffered channel |

## Registered Job Types

| Job Type | Module | Handler | Description |
| -------- | ------ | ------- | ----------- |
| `term_scan` | [[Palimpsest-Writing-Software/features/glossary/overview\|Glossary]] | `glossary.TermScanHandler` | Scan content and apply/remove `glossaryLink` ProseMirror marks for a glossary entry |
| `search_reindex` | [[Palimpsest-Writing-Software/features/search/overview\|Search]] | `search.ReindexHandler` | Drop and rebuild all FTS5 search index entries for a project |
| `full_rescan` | — | — | Reserved enum value for future full-project rescan (not yet implemented) |

## Architecture

### Handler Registration Pattern

Modules register their handlers during server startup in `main.go`. Registration is fail-fast: duplicate job type strings cause a panic to catch wiring errors immediately.

```
main.go startup:
  1. jobManager = jobs.NewManager(store)
  2. jobManager.RegisterHandler(termScanHandler)    // "term_scan"
  3. jobManager.RegisterHandler(reindexHandler)      // "search_reindex"
  4. Server receives jobManager as optional parameter
```

### Per-Project Worker Goroutines

The Manager creates **one worker goroutine per project** the first time a job is submitted for that project. This design avoids SQLite contention — since each project has its own `.db` file with a single-writer lock, sequential per-project execution prevents write conflicts.

Each worker pulls jobs from a **buffered channel** (capacity: 100) and processes them one at a time. If the channel is full when a new job is submitted, the job record still exists in `"pending"` state in the database — it will be picked up when the worker drains its queue.

### Job Lifecycle

```
┌─────────┐     ┌─────────┐     ┌───────────┐
│ pending  │ ──► │ running  │ ──► │ completed │
└─────────┘     └─────────┘     └───────────┘
                     │
                     ├──────────► ┌────────┐
                     │            │ failed │
                     │            └────────┘
                     │
                     └──────────► ┌───────────┐
                                  │ cancelled │
                                  └───────────┘
```

| Status | Meaning | Timing Fields Set |
| ------ | ------- | ----------------- |
| `pending` | Job created, waiting for worker | `created_at` |
| `running` | Worker picked up job, handler executing | `started_at` |
| `completed` | Handler returned nil, job succeeded | `completed_at` |
| `failed` | Handler returned error | `completed_at`, `error_message` |
| `cancelled` | User requested cancellation via API | `completed_at` |

### Callback-Based Decoupling

Job handlers use **callback functions** (not direct repository imports) for data access. This keeps modules decoupled from the repository layer:

- `glossary.TermScanHandler` receives `EntryLoader`, `ContentLoader`, `ContentBodySaver`, `CountUpdater` callbacks
- `search.ReindexHandler` receives `ContentLoader`, `GlossaryLoader` callbacks

The callbacks are wired in `main.go` where the repository is available, but the handler packages never import `repository` directly.

## Data Flow

### Job Submission

1. Module-specific HTTP handler receives user action (e.g., glossary entry save)
2. Handler calls `jobManager.Submit(ctx, projectSlug, jobType, targetID, params)`
3. Manager validates a handler exists for the job type
4. Manager calls `store.CreateJob()` to persist a `"pending"` record
5. Manager ensures a worker goroutine exists for the project (`ensureWorker`)
6. Job request is sent to the project's buffered channel
7. Manager returns the job ID to the HTTP handler for status polling

### Job Execution

1. Worker goroutine receives job request from channel
2. Manager calls `store.UpdateJobStatus()` → `"running"`, sets `started_at`
3. Manager builds a `JobContext` with `ProgressReporter`
4. Manager calls `handler.Execute(ctx, jobCtx)`
5. Handler processes items, periodically calling `progress.Update(processed, matched)`
6. Handler checks `progress.IsCancelled()` between processing steps
7. On success: Manager marks `"completed"`, sets `completed_at`
8. On error: Manager marks `"failed"`, sets `completed_at` and `error_message`
9. On cancellation: Status already set via API; handler returns early

### Job Status Polling

1. Frontend submits job via module endpoint → receives job ID
2. Frontend polls `GET /api/projects/{slug}/jobs/{id}` periodically
3. Handler returns `JobResponse` with current status, progress counts, timestamps
4. Frontend uses progress data to update UI (progress bars, notifications)
5. Polling stops when status is terminal (`completed`, `failed`, `cancelled`)

## Database Schema

Jobs are stored using the `ScanJob` Ent schema in each project's SQLite database:

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `id` | `string` | UUID v4 | Unique job identifier |
| `job_type` | `enum` | `"term_scan"` | Job type: `term_scan`, `full_rescan`, `search_reindex` |
| `status` | `enum` | `"pending"` | Lifecycle status: `pending`, `running`, `completed`, `failed`, `cancelled` |
| `target_id` | `string?` | — | Module-specific target (glossary entry ID, etc.) |
| `total_items` | `int` | `0` | Total items to process |
| `processed_items` | `int` | `0` | Items processed so far |
| `matched_items` | `int` | `0` | Items with matches found |
| `error_message` | `string?` | — | Error details if failed |
| `started_at` | `time?` | — | When processing began |
| `completed_at` | `time?` | — | When processing ended |
| `created_at` | `time` | `now()` | Record creation timestamp |
| `updated_at` | `time` | `now()` | Last update timestamp |

### Indexes

| Fields | Purpose |
| ------ | ------- |
| `target_id`, `status` | Find active jobs for a specific target (duplicate prevention) |
| `status` | Find all jobs by status (monitoring, cleanup) |
| `created_at` | Find recent jobs (listing, pagination) |

## Security

- **No direct frontend submission** — Jobs are submitted by backend handlers, not by frontend API calls. The glossary handler submits `term_scan` jobs; the search handler submits `search_reindex` jobs. The jobs API only exposes status and cancellation.
- **Project-scoped access** — All job operations require a `projectSlug` path parameter. Jobs belong to specific projects and cannot be accessed cross-project.
- **Cancellation guard** — Only `pending` or `running` jobs can be cancelled. Terminal states (`completed`, `failed`, `cancelled`) reject cancellation attempts with a 400 error.
- **Handler validation** — `Submit()` rejects job types with no registered handler, preventing orphaned job records.
- **Fail-fast registration** — Duplicate handler registration panics during startup, catching wiring errors before the server accepts requests.

## Performance

- **Per-project workers** — One worker goroutine per project avoids SQLite single-writer contention. Multiple projects process jobs concurrently.
- **Buffered channels** — 100-slot buffered channel per project allows non-blocking submission even when the worker is busy.
- **Sequential execution** — Jobs within a project run sequentially, which is correct for SQLite but may need a pool for future PostgreSQL mode.
- **Progress batching** — Handlers call `progress.Update()` periodically (not per-item) to reduce database write frequency.
- **Cancellation polling** — `IsCancelled()` queries the database for current status. Handlers should check between processing steps, not per-item, to minimize overhead.
- **Graceful shutdown** — `Shutdown(ctx)` closes the shutdown channel and waits for in-flight jobs via `sync.WaitGroup`, with context-based timeout.

## Design Decisions

| Decision | Rationale |
| -------- | --------- |
| One worker per project | SQLite uses a single-writer lock per `.db` file. Parallel jobs within the same project would cause contention. Sequential execution is correct for the local-first architecture. |
| Handler registration at startup | Fail-fast validation ensures all job types are wired before the server accepts requests. Runtime registration would risk orphaned jobs. |
| Callback-based data access | Handlers receive data loader/saver functions instead of importing the repository package directly. This keeps module packages decoupled and testable. |
| Buffered channel (100 slots) | Allows burst submission without blocking. Jobs exceeding the buffer still have database records in `"pending"` state, though they won't be processed until the channel drains. |
| No job listing endpoint | Current design only supports get-by-ID and cancel. Listing would require additional query patterns and pagination. The `created_at` index is in place for future listing support. |
| Status via database polling | `IsCancelled()` queries the database rather than using in-memory signals. This survives server restarts and works correctly when the cancellation request arrives on a different goroutine. |
| `ScanJob` entity (not in-memory only) | Persisting jobs to the database enables status polling across HTTP requests and survives server restarts. In-memory-only jobs would be lost on crash. |

## Stubs and Future Work

| Item | Status | Notes |
| ---- | ------ | ----- |
| `full_rescan` job type | Enum defined | No handler registered; reserved for future full-project rescan |
| Job listing endpoint | Not implemented | `GET /api/projects/{slug}/jobs` returns 405; `created_at` index ready |
| `search_reindex` content/glossary loaders | Stub | `NewReindexHandler` created with `nil` loaders; wired to real data when list-all methods are added |
| PostgreSQL job concurrency | Not started | Per-project sequential workers are correct for SQLite; may need worker pool for PostgreSQL mode |
| Job retry/backoff | Not implemented | Failed jobs stay in `"failed"` state; no automatic retry mechanism |
| Job cleanup/expiry | Not implemented | Old completed/failed jobs accumulate; no TTL or cleanup cron |

## Diagrams

- [[Palimpsest-Writing-Software/architecture/jobs/diagrams/job-submit-execute-flow\|Job Submit and Execute: Full Lifecycle]]
- [[Palimpsest-Writing-Software/architecture/jobs/diagrams/job-handler-dispatch-flow\|Job Handler Dispatch: Registration to Execution]]

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/api/jobs/overview\|Jobs API Service]]
- [[Palimpsest-Writing-Software/features/glossary/overview\|Glossary System]]
- [[Palimpsest-Writing-Software/features/search/overview\|Search Engine]]
- [[Palimpsest-Writing-Software/architecture/logging/overview\|Logging System]]
