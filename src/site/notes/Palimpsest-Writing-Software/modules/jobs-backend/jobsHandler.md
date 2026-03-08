---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/jobs-backend/jobs-handler/","title":"jobs.go — HTTP Handler","tags":["file","jobs-backend"],"updated":"2026-03-05T09:15:40.314-07:00"}
---


# `jobs.go`

**Module:** `[[modules/jobs-backend/overview|Jobs Backend]]`
**Path:** `backend/internal/http/handlers/jobs.go`

> [!NOTE] HTTP handler providing REST endpoints for job status queries and cancellation. Job submission is handled by module-specific endpoints (e.g., glossary scan triggers a `term_scan` job); this handler only exposes status and cancel operations.

## Purpose

This file defines the `JobHandler` HTTP handler struct and its route dispatcher. It translates HTTP requests into calls on `[[modules/jobs-backend/manager|Manager]]` and converts `[[modules/jobs-backend/types#JobDTO|JobDTO]]` responses into JSON with camelCase field names and ISO 8601 timestamps. It supports graceful degradation: if the `Manager` is nil (e.g., job system not configured), all endpoints return 503 Service Unavailable.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `JobHandler` | struct | HTTP handler for job operations |
| `NewJobHandler` | constructor | Creates handler; nil manager causes 503 on all endpoints |
| `JobResponse` | struct | API response DTO with camelCase JSON tags |
| `JobHandler.Jobs` | method | Route dispatcher for `/api/projects/{slug}/jobs/...` |

---

## `JobHandler` Struct

| Field | Type | Access | Description |
| ----- | ---- | ------ | ----------- |
| `manager` | `*jobs.Manager` | private | Job manager for submitting and querying jobs; may be nil |
| `log` | `*logging.Logger` | private | Logger with component `"handler-jobs"` |

---

## `JobResponse` Struct

API response format for job status queries. All timestamps are formatted as ISO 8601 strings (`2006-01-02T15:04:05Z`).

| Field | Type | JSON Tag | Description |
| ----- | ---- | -------- | ----------- |
| `ID` | `string` | `"id"` | Job UUID |
| `JobType` | `string` | `"jobType"` | Job type string |
| `Status` | `string` | `"status"` | Current status |
| `TargetID` | `string` | `"targetId,omitempty"` | Module-specific target ID |
| `TotalItems` | `int` | `"totalItems"` | Total items to process |
| `ProcessedItems` | `int` | `"processedItems"` | Items processed so far |
| `MatchedItems` | `int` | `"matchedItems"` | Items with matches |
| `ErrorMessage` | `string` | `"errorMessage,omitempty"` | Error details (failed jobs only) |
| `StartedAt` | `*string` | `"startedAt,omitempty"` | ISO 8601 start time (nil if pending) |
| `CompletedAt` | `*string` | `"completedAt,omitempty"` | ISO 8601 completion time (nil if not terminal) |
| `CreatedAt` | `string` | `"createdAt"` | ISO 8601 creation time |
| `UpdatedAt` | `string` | `"updatedAt"` | ISO 8601 last update time |

---

## Functions

### `NewJobHandler(manager *jobs.Manager) *JobHandler`

Creates a new `JobHandler`. If `manager` is nil, all endpoints return 503 Service Unavailable. Logging is initialized with `logging.GetGlobal().WithComponent("handler-jobs")`.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `manager` | `*jobs.Manager` | no | Job manager (nil causes 503 degradation) |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Always | Initialized handler | `*JobHandler` |

---

### `Jobs(w http.ResponseWriter, r *http.Request)`

Main route dispatcher for `/api/projects/{slug}/jobs/...`. Extracts the project slug from request context and the job ID from the URL path, then dispatches by HTTP method.

**Routing logic:**

| Path Pattern | Method | Handler | Description |
| ------------ | ------ | ------- | ----------- |
| `/api/projects/{slug}/jobs` | any | — | Returns 405 Method Not Allowed (listing not yet implemented) |
| `/api/projects/{slug}/jobs/{id}` | `GET` | `getJob` | Returns job status |
| `/api/projects/{slug}/jobs/{id}` | `DELETE` | `cancelJob` | Cancels the job |
| `/api/projects/{slug}/jobs/{id}` | other | — | Returns 405 Method Not Allowed |

> [!WARNING] Job listing (`GET /api/projects/{slug}/jobs` without an ID) returns 405 Method Not Allowed. This endpoint is not yet implemented.

---

### `getJob(w, r, projectSlug, jobID string)`

Handles `GET /api/projects/{slug}/jobs/{id}`. Fetches job status from the `[[modules/jobs-backend/manager|Manager]]` and returns a `JobResponse` as JSON.

| Condition | HTTP Status | Response |
| --------- | ----------- | -------- |
| Success | 200 OK | `JobResponse` JSON |
| Job not found | 404 Not Found | `{"error": "job not found"}` |
| Internal error | 500 Internal Server Error | `{"error": "failed to get job: ..."}` |

---

### `cancelJob(w, r, projectSlug, jobID string)`

Handles `DELETE /api/projects/{slug}/jobs/{id}`. Delegates to `[[modules/jobs-backend/manager|Manager.CancelJob()]]`.

| Condition | HTTP Status | Response |
| --------- | ----------- | -------- |
| Success | 204 No Content | Empty body |
| Job not found | 404 Not Found | `{"error": "job not found"}` |
| Job already terminal | 400 Bad Request | `{"error": "jobs: cannot cancel job in status \"...\""}` |
| Internal error | 500 Internal Server Error | `{"error": "failed to cancel job: ..."}` |

---

### `extractJobID(fullPath, projectSlug string) string`

Parses the job ID from the URL path. Strips the prefix `/api/projects/{slug}/jobs/` and returns the next path segment. Returns an empty string if no job ID is present (i.e., the request is for the jobs collection endpoint).

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `fullPath` | `string` | yes | Full URL path from `r.URL.Path` |
| `projectSlug` | `string` | yes | Project slug for prefix construction |

---

### `jobDTOToResponse(job *jobs.JobDTO) JobResponse`

Converts a `[[modules/jobs-backend/types#JobDTO|JobDTO]]` to a `JobResponse` with ISO 8601 formatted timestamp strings. Nullable `time.Time` pointers (`StartedAt`, `CompletedAt`) are converted to nullable `*string` pointers.

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `job` | `*jobs.JobDTO` | yes | Job DTO from the store/manager layer |

| Condition | Returns | Type |
| --------- | ------- | ---- |
| Always | Formatted response | `JobResponse` |

---

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `net/http` | stdlib | HTTP handler types |
| `strings` | stdlib | Error message matching and URL path parsing |
| `contextx` | internal | `GetProjectSlugFromRequest()` for extracting project slug from request context |
| `response` | internal | HTTP response helpers (`OK`, `BadRequest`, `NotFound`, `InternalError`, `NoContent`, `MethodNotAllowed`) |
| `jobs` | internal | `Manager`, `JobDTO` types |
| `logging` | internal | `WithComponent("handler-jobs")` for structured logging |

## Side Effects

None. This handler is stateless and delegates all persistence to the `[[modules/jobs-backend/manager|Manager]]`.

## Notes

> [!TIP] Error detection in `getJob` and `cancelJob` uses `strings.Contains(err.Error(), "not found")` to distinguish 404 responses from 500 responses. This matches the error format used by `[[modules/jobs-backend/scanJobStore|scan_job.go]]` (e.g., `"scan job not found: {id}"`).

## Related

- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend Module]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/manager\|manager.go — Job Manager]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/types\|types.go — JobDTO, JobResponse]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/scanJobStore\|scan_job.go — Error Format]]
- [[Palimpsest-Writing-Software/api/jobs/overview\|Jobs API Service]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System Architecture]]
