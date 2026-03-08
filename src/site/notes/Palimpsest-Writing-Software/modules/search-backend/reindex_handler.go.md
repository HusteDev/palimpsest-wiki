---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-backend/reindex-handler-go/","title":"search/reindex_handler.go","tags":["file","search-backend"],"updated":"2026-03-05T05:32:44.530-07:00"}
---


# `search/reindex_handler.go`

**Module:** [[Palimpsest-Writing-Software/modules/search-backend/overview\|Search Backend]]
**Path:** `backend/internal/search/reindex_handler.go`

> [!NOTE] Background job handler for the `search_reindex` job type. Rebuilds the complete FTS5 search index for a project by loading all content and glossary data via callbacks, clearing existing indexes, and re-inserting everything.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ReindexHandler` | struct | Implements `jobs.JobHandler` for `search_reindex` jobs |
| `NewReindexHandler` | function | Constructor: `(provider, contentLoader, glossaryLoader)` returns `*ReindexHandler` |

### ReindexHandler Struct

```go
type ReindexHandler struct {
    provider       SearchProvider  // The search provider to use for indexing
    contentLoader  ContentLoader   // Callback to load all content for a project
    glossaryLoader GlossaryLoader  // Callback to load all glossary entries for a project
    log            *logging.Logger // Injected logger for this handler instance
}
```

### Loader Callback Types

Defined in `types.go` to avoid circular imports with the repository package:

```go
type ContentLoader func(ctx context.Context, projectSlug string) ([]ContentIndexData, error)
type GlossaryLoader func(ctx context.Context, projectSlug string) ([]GlossaryIndexData, error)
```

```go
type ContentIndexData struct {
    ID        string // Content entity UUID
    Title     string // Document title
    PlainText string // Plain text extracted from ProseMirror JSON
    Tags      string // Comma-separated tag list
    SlugPath  string // Full slug path (e.g. "series-1/book-1/chapter-3")
}

type GlossaryIndexData struct {
    ID           string // Glossary entry UUID
    Term         string // Primary term
    Definition   string // Entry definition
    Aliases      string // Comma-separated alias list
    Tags         string // Comma-separated tag list
    ScopeTargets string // Comma-separated scope target paths (empty until glossary-v2)
}
```

## Key Methods

### JobType

```
JobType() string
```

Returns the job type string `"search_reindex"`. Must match the enum value in the ScanJob schema so the job manager routes reindex jobs to this handler.

**Returns:** `"search_reindex"`

### Execute

```
Execute(ctx context.Context, job jobs.JobContext) error
```

Runs the full reindex operation for the project specified in the job. This is the main entry point called by the job manager when a `search_reindex` job is dequeued.

**Execution flow:**

1. Load all content data via `contentLoader(ctx, job.ProjectSlug)`
2. Load all glossary data via `glossaryLoader(ctx, job.ProjectSlug)`
3. Report initial progress: `job.Progress.Update(0, 0)`
4. Create cached loader wrappers that return the already-loaded data (avoids double-loading when `provider.ReindexAll` calls them)
5. Delegate to `provider.ReindexAll` which clears both FTS5 tables and re-inserts all entries
6. Check for job cancellation via `job.Progress.IsCancelled()`
7. Report completion: `job.Progress.Update(totalItems, totalItems)`

**Parameters:**
- `ctx` -- context for cancellation/timeout
- `job` -- `jobs.JobContext` containing `JobID`, `ProjectSlug`, `Progress` reporter, and other job metadata

**Returns:** `nil` on success, error if content/glossary loading fails or `ReindexAll` fails

**Progress reporting:**
- Initial: `Update(0, 0)` signals the job has started
- Final: `Update(totalItems, totalItems)` where `totalItems = len(content) + len(glossary)`
- Cancellation is checked after `ReindexAll` completes; if cancelled, the handler returns `nil` without error

## Registration Pattern

```go
handler := search.NewReindexHandler(searchProvider, contentLoader, glossaryLoader)
jobManager.RegisterHandler(handler)
```

The handler is registered with the job manager at application startup in `backend/cmd/server/main.go`. The `contentLoader` and `glossaryLoader` callbacks are typically wired to repository methods that enumerate all indexable data for a project.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `jobs` | `backend/internal/jobs` | `JobHandler` interface and `JobContext` type |
| `logging` | `backend/internal/logging` | Structured logging with `"search-reindex"` component |
| `context` | stdlib | Request context for cancellation/timeout |
| `fmt` | stdlib | Error wrapping |

## Side Effects

- Clears and rebuilds FTS5 virtual tables in the per-project SQLite database (via `provider.ReindexAll`)
- Reports progress to the job manager via `job.Progress.Update()`
- Logs at INFO level for start/completion and data counts

## Notes

> [!WARNING] The handler loads ALL content and glossary data into memory before indexing. For very large projects this could consume significant memory. The cached loader pattern avoids double-loading but does not stream data. If memory becomes a concern, the loader callbacks and `ReindexAll` would need to be refactored to support streaming/batching.
