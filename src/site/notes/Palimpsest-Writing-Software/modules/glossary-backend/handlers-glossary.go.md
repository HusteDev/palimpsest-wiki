---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-backend/handlers-glossary-go/","title":"handlers/glossary.go","tags":["file","glossary-backend"],"updated":"2026-03-05T05:32:42.489-07:00"}
---


# `handlers/glossary.go`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Glossary Backend]]
**Path:** `backend/internal/http/handlers/glossary.go`

> [!NOTE] HTTP handler for all glossary API endpoints. Dispatches requests by path and HTTP method, validates input, calls the repository, and submits background term scan jobs.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `GlossaryHandler` | struct | HTTP handler with store, search, jobs, and logger fields |
| `NewGlossaryHandler` | function | Constructor: `(store, searchProvider, jobSubmitter)` returns `*GlossaryHandler` |

### GlossaryHandler struct

```go
type GlossaryHandler struct {
    store  repository.ProjectStore
    search search.SearchProvider   // may be nil
    jobs   JobSubmitter            // may be nil
    log    *logging.Logger
}
```

### JobSubmitter interface

```go
type JobSubmitter interface {
    Submit(ctx context.Context, projectSlug, jobType, targetID string, params map[string]any) (string, error)
}
```

Matches the `*jobs.Manager.Submit()` signature. Defined locally to keep the handler package decoupled from the jobs package.

## Key Methods

| Method | Description |
| ------ | ----------- |
| `Glossary(w, r)` | Dispatcher -- extracts `projectSlug` from request context, parses path after `/glossary`, and routes to the appropriate handler method by HTTP method |
| `createEntry(w, r, slug)` | **POST** -- validates required `term` and `shortDefinition`, applies defaults (`autoMark=true`, `enableTooltip=true`, `enableHyperlink=false`), calls `store.CreateGlossaryEntry`, submits `term_scan` job with action `"scan"`, returns `201 Created` |
| `getEntry(w, r, slug, id)` | **GET** -- calls `store.GetGlossaryEntry`, returns `200 OK` or `404 Not Found` |
| `listEntries(w, r, slug)` | **GET** -- parses query params (`limit` 1-100, `offset`, `search`, `tags`, `category`, `status`, `sortBy`, `sortOrder`), calls `store.ListGlossaryEntries`, returns paginated response |
| `updateEntry(w, r, slug, id)` | **PUT** -- validates at least one field is non-nil, calls `store.UpdateGlossaryEntry`, submits `term_scan` job with action `"scan"`, returns `200 OK` |
| `deleteEntry(w, r, slug, id)` | **DELETE** -- calls `store.DeleteGlossaryEntry`, submits `term_scan` job with action `"remove"`, returns `204 No Content` |
| `getOccurrences(w, r, slug, id)` | **GET** -- loads entry, searches FTS5 for the term across content documents, builds occurrence response with `scanPending` flag (true when FTS5 found docs but marks have not been applied yet) |
| `submitTermScan(ctx, slug, id, action)` | Helper -- submits a `term_scan` background job; no-op if `jobs` is nil (graceful degradation) |
| `glossaryEntryToResponse(dto)` | Helper -- converts `repository.GlossaryEntryDTO` to the API JSON response struct with RFC3339-formatted timestamps |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `repository.ProjectStore` | `backend/internal/repository` | CRUD operations for glossary entries |
| `search.SearchProvider` | `backend/internal/search` | FTS5 occurrence search for `getOccurrences` |
| `logging.Logger` | `backend/internal/logging` | Structured logging scoped to `"handler-glossary"` component |
| `response` package | `backend/internal/http/response` | HTTP response helpers (`OK`, `Created`, `NoContent`, `BadRequest`, etc.) |
| `request` package | `backend/internal/http/request` | Query param parsing and safe JSON body deserialization |
| `contextx` package | `backend/internal/http/contextx` | Extract `projectSlug` from request context |
| `encoding/json` | stdlib | JSON decoding of request bodies |
| `net/http` | stdlib | HTTP types and method constants |
| `strconv` | stdlib | Integer parsing for query parameters |
| `strings` | stdlib | String manipulation for path parsing and validation |

## Side Effects

- Submits background `term_scan` jobs after create, update, and delete operations. The job runs asynchronously via the job manager and applies or removes ProseMirror marks in content documents.
- Reads and writes glossary entries via `ProjectStore`. All database operations go through the repository interface.
- Reads content via `SearchProvider.Search()` for the occurrences endpoint.

## Notes

> [!WARNING] The `getOccurrences` endpoint calculates `scanPending` heuristically: if the entry's stored `usageCount` is 0 but FTS5 found matching documents and `autoMark` is enabled, it assumes the term scan has not completed yet. This may briefly show false positives if a term genuinely has no marks.
