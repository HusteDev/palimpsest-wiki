---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/jobs/overview/","title":"Jobs API Service","tags":["api","jobs"],"updated":"2026-03-05T09:15:23.589-07:00"}
---


# Jobs API Service

> [!NOTE]
> REST API for querying background job status and cancelling pending or running jobs. Job *submission* is not part of this API — modules submit jobs through their own endpoints.

## Base URL

```
/api/projects/{slug}/jobs
```

## Handler

[[Palimpsest-Writing-Software/modules/jobs-backend/jobsHandler\|JobHandler]]

## Authentication

None (local desktop application).

## Response Format

JSON with camelCase keys.

## Overview

The Jobs API is a **status-and-cancel** service. It does not accept job submissions directly. Instead, each module that needs background work submits jobs through its own endpoint (see [[Palimpsest-Writing-Software/api/jobs/overview#Integration Points\|#Integration Points]] below). Once a job exists, any client can poll its status or cancel it through this API.

If the server was created without a job manager (i.e., the `manager` field is `nil`), **all** jobs endpoints return `503 Service Unavailable`.

Job listing (`GET /api/projects/{slug}/jobs`) is not yet implemented and currently returns `405 Method Not Allowed`.

## Routes

| Method   | Path          | Description                                  | Status          |
| -------- | ------------- | -------------------------------------------- | --------------- |
| GET      | `/jobs/{id}`  | Retrieve current status of a background job  | Implemented     |
| DELETE   | `/jobs/{id}`  | Cancel a pending or running job              | Implemented     |
| GET      | `/jobs`       | List all jobs for a project                  | Not Implemented |

All paths are relative to the base URL `/api/projects/{slug}`.

---

## `GET /api/projects/{slug}/jobs/{id}`

**Handler:** `[[modules/jobs-backend/jobsHandler#getJob]]`

> [!NOTE]
> Retrieve the current status and progress counters for a single background job.

### Request

**Auth required:** None

#### Path Parameters

| Parameter | Type   | Description                            |
| --------- | ------ | -------------------------------------- |
| `slug`    | string | Project slug identifying the project   |
| `id`      | UUID   | Unique identifier of the job to query  |

#### Query Parameters

None.

#### Request Body

None.

### Response

#### Success --- `200 OK`

Returns a `JobResponse` object:

```json
{
  "id": "string (UUID)",
  "jobType": "string (term_scan | search_reindex | full_rescan)",
  "status": "string (pending | running | completed | failed | cancelled)",
  "targetId": "string? (module-specific target ID)",
  "totalItems": 0,
  "processedItems": 0,
  "matchedItems": 0,
  "errorMessage": "string? (only present when status=failed)",
  "startedAt": "string? (ISO 8601, set when status transitions to running)",
  "completedAt": "string? (ISO 8601, set when status reaches a terminal state)",
  "createdAt": "string (ISO 8601)",
  "updatedAt": "string (ISO 8601)"
}
```

##### Field Reference

| Field            | Type    | Nullable | Description                                                        |
| ---------------- | ------- | -------- | ------------------------------------------------------------------ |
| `id`             | string  | no       | UUID of the job                                                    |
| `jobType`        | string  | no       | One of `term_scan`, `search_reindex`, `full_rescan`                |
| `status`         | string  | no       | One of `pending`, `running`, `completed`, `failed`, `cancelled`    |
| `targetId`       | string  | yes      | Module-specific target (e.g., glossary entry ID for `term_scan`)   |
| `totalItems`     | int     | no       | Total items the job will process                                   |
| `processedItems` | int     | no       | Items processed so far                                             |
| `matchedItems`   | int     | no       | Items that matched the job criteria (e.g., term occurrences found) |
| `errorMessage`   | string  | yes      | Human-readable error detail; only present when `status=failed`     |
| `startedAt`      | string  | yes      | ISO 8601 timestamp; set when the job begins execution              |
| `completedAt`    | string  | yes      | ISO 8601 timestamp; set when the job reaches a terminal state      |
| `createdAt`      | string  | no       | ISO 8601 timestamp of job creation                                 |
| `updatedAt`      | string  | no       | ISO 8601 timestamp of last status update                           |

#### Error Responses

| Status | Condition                            | Body                                    |
| ------ | ------------------------------------ | --------------------------------------- |
| `404`  | Job with the given ID was not found  | `{ "error": "job not found" }`          |
| `500`  | Internal server error                | `{ "error": "failed to get job: ..." }` |
| `503`  | Job manager not configured on server | `{ "error": "job manager not configured" }` |

---

## `DELETE /api/projects/{slug}/jobs/{id}`

**Handler:** `[[modules/jobs-backend/jobsHandler#cancelJob]]`

> [!NOTE]
> Cancel a job that is in the `pending` or `running` state. Jobs that have already reached a terminal state (`completed`, `failed`, `cancelled`) cannot be cancelled.

### Request

**Auth required:** None

#### Path Parameters

| Parameter | Type   | Description                             |
| --------- | ------ | --------------------------------------- |
| `slug`    | string | Project slug identifying the project    |
| `id`      | UUID   | Unique identifier of the job to cancel  |

#### Query Parameters

None.

#### Request Body

None.

### Response

#### Success --- `204 No Content`

Empty response body. The job has been marked as cancelled.

#### Error Responses

| Status | Condition                                                          | Body                                        |
| ------ | ------------------------------------------------------------------ | ------------------------------------------- |
| `400`  | Job is in a terminal state (`completed`, `failed`, or `cancelled`) | `{ "error": "cannot cancel job: ..." }`     |
| `404`  | Job with the given ID was not found                                | `{ "error": "job not found" }`              |
| `500`  | Internal server error                                              | `{ "error": "failed to cancel job: ..." }`  |
| `503`  | Job manager not configured on server                               | `{ "error": "job manager not configured" }` |

---

## Job Status Lifecycle

Jobs progress through the following states:

```
pending --> running --> completed
                   \-> failed
           \-> cancelled (via DELETE)
```

| Status      | Terminal | Description                                      |
| ----------- | -------- | ------------------------------------------------ |
| `pending`   | no       | Job is queued but has not started execution       |
| `running`   | no       | Job is currently being processed                  |
| `completed` | yes      | Job finished successfully                         |
| `failed`    | yes      | Job encountered an error; see `errorMessage`      |
| `cancelled` | yes      | Job was cancelled via the DELETE endpoint          |

Only `pending` and `running` jobs can be cancelled. Attempting to cancel a terminal job returns `400`.

## Integration Points

The jobs API does not accept job submissions. Jobs are created by module-specific handlers:

### Glossary Module

The [[Palimpsest-Writing-Software/api/glossary/overview\|Glossary API]] submits `term_scan` jobs when glossary entries are created, updated, or deleted. The `targetId` field on these jobs contains the glossary entry ID that triggered the scan.

- **Create entry** (`POST /glossary`) --- submits a `term_scan` job for the new entry
- **Update entry** (`PUT /glossary/{id}`) --- submits a `term_scan` job for the updated entry
- **Delete entry** (`DELETE /glossary/{id}`) --- submits a `term_scan` job to clean up occurrences

### Search Module

The [[Palimpsest-Writing-Software/api/search/overview\|Search API]] submits `search_reindex` jobs when a full index rebuild is requested. These jobs scan all project content to rebuild the search index.

### Job Types

| Job Type         | Submitting Module | Purpose                                           |
| ---------------- | ----------------- | ------------------------------------------------- |
| `term_scan`      | Glossary          | Scan content for occurrences of a glossary term   |
| `search_reindex` | Search            | Rebuild the full-text search index from scratch   |
| `full_rescan`    | (reserved)        | Full project rescan across all modules            |

## Related

- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Architecture: Jobs System]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Module: Jobs Backend]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/jobsHandler\|Source: JobHandler]]
- [[Palimpsest-Writing-Software/api/glossary/overview\|API: Glossary Service]]
- [[Palimpsest-Writing-Software/api/search/overview\|API: Search Service]]
