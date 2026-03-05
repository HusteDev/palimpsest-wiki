---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-backend/fts5-go/","title":"search/fts5.go","tags":["file","search-backend"]}
---


# `search/fts5.go`

**Module:** [[Palimpsest-Writing-Software/modules/search-backend/overview\|Search Backend]]
**Path:** `backend/internal/search/fts5.go`

> [!NOTE] FTS5-based implementation of the `SearchProvider` interface. Manages SQLite FTS5 virtual tables for content and glossary indexing, BM25-ranked searching with snippet highlighting, LIKE fallback for suffix/infix wildcards, and full reindexing. Also covers `fts5_schema.go` (DDL definitions) and `stub.go` (no-op fallback).

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SearchProvider` | interface | Backend-agnostic search abstraction (7 methods) defined in `types.go` |
| `FTS5Provider` | struct | SQLite FTS5 implementation of `SearchProvider` |
| `NewFTS5Provider` | function | Constructor: `(openDB DBOpener)` returns `*FTS5Provider` |
| `StubProvider` | struct | No-op fallback implementation (graceful degradation) |
| `NewStubProvider` | function | Constructor: returns `*StubProvider` |
| `RunEnsureIndexes` | function | Creates FTS5 virtual tables if not present (idempotent) |
| `ContentFTSCreate` | const | SQL DDL for `content_fts` virtual table |
| `GlossaryFTSCreate` | const | SQL DDL for `glossary_fts` virtual table |

### SearchProvider Interface

```go
type SearchProvider interface {
    IndexContent(ctx context.Context, projectSlug string, id string, fields map[string]string) error
    IndexGlossaryEntry(ctx context.Context, projectSlug string, id string, fields map[string]string) error
    RemoveFromIndex(ctx context.Context, projectSlug string, indexType string, id string) error
    Search(ctx context.Context, projectSlug string, query string, opts SearchOptions) (*SearchResult, error)
    FindDocumentsContaining(ctx context.Context, projectSlug string, terms []string) ([]string, error)
    ReindexAll(ctx context.Context, projectSlug string, contentLoader ContentLoader, glossaryLoader GlossaryLoader) error
    EnsureIndexes(ctx context.Context, projectSlug string) error
}
```

All methods accept a `projectSlug` so the provider can resolve the correct per-project database internally.

### FTS5Provider Struct

```go
type FTS5Provider struct {
    openDB DBOpener        // Injected function to open a per-project *sql.DB connection
    log    *logging.Logger // Injected logger for this provider instance
}
```

### DBOpener Type

```go
type DBOpener func(ctx context.Context, projectSlug string) (*sql.DB, error)
```

Injected at init to decouple the search package from the repository/sqlite package.

### StubProvider Struct

```go
type StubProvider struct {
    warnOnce sync.Once       // Ensures the "search unavailable" warning logs only once
    log      *logging.Logger // Injected logger
}
```

All index operations return `nil` (success). `Search` returns empty results with a one-time warning log.

## FTS5 Schema

Defined in `fts5_schema.go`. Ent ORM does not support SQLite virtual tables, so these are created via raw SQL.

### content_fts Table

```sql
CREATE VIRTUAL TABLE IF NOT EXISTS content_fts USING fts5(
    content_id UNINDEXED,   -- UUID for joining back to contents table (not tokenized)
    title,                   -- Document title (searchable)
    plain_text,              -- Plain text from ProseMirror JSON (searchable)
    tags,                    -- Comma-separated tag list (searchable)
    slug_path UNINDEXED      -- Full slug path for display (not tokenized)
)
```

### glossary_fts Table

```sql
CREATE VIRTUAL TABLE IF NOT EXISTS glossary_fts USING fts5(
    entry_id UNINDEXED,      -- UUID for joining back to glossary entries (not tokenized)
    term,                     -- Primary glossary term (searchable)
    definition,               -- Entry definition text (searchable)
    aliases,                  -- Comma-separated alias list (searchable)
    tags,                     -- Comma-separated tag list (searchable)
    scope_targets UNINDEXED   -- Comma-separated scope paths for post-filtering (not tokenized)
)
```

`UNINDEXED` columns are stored in the FTS5 table but excluded from the full-text index. They are used for filtering and display without affecting search ranking.

## Key Methods

### EnsureIndexes

```
EnsureIndexes(ctx context.Context, projectSlug string) error
```

Creates both FTS5 virtual tables if they do not already exist. Delegates to `RunEnsureIndexes` which executes the `ContentFTSCreate` and `GlossaryFTSCreate` DDL constants. Idempotent -- safe to call on every database open. Called from `openEnt()` in the SQLite store after Ent schema migration completes.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug used to open the database

**Returns:** error if table creation fails

### IndexContent

```
IndexContent(ctx context.Context, projectSlug string, id string, fields map[string]string) error
```

Adds or replaces a content document in the FTS5 index. Uses the DELETE + INSERT upsert pattern: deletes any existing entry with the same `content_id`, then inserts the new row. The delete failure is non-fatal (warns and continues) since the row may not exist yet.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug
- `id` -- content entity UUID
- `fields` -- map with keys: `"title"`, `"plain_text"`, `"tags"`, `"slug_path"`

**Returns:** error if the INSERT fails

### IndexGlossaryEntry

```
IndexGlossaryEntry(ctx context.Context, projectSlug string, id string, fields map[string]string) error
```

Adds or replaces a glossary entry in the FTS5 index. Same DELETE + INSERT upsert pattern as `IndexContent`.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug
- `id` -- glossary entry UUID
- `fields` -- map with keys: `"term"`, `"definition"`, `"aliases"`, `"tags"`, `"scope_targets"`

**Returns:** error if the INSERT fails

### RemoveFromIndex

```
RemoveFromIndex(ctx context.Context, projectSlug string, indexType string, id string) error
```

Removes a single item from the FTS5 index.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug
- `indexType` -- `"content"` or `"glossary"` (determines which FTS5 table)
- `id` -- entity UUID to remove

**Returns:** error if the DELETE fails or `indexType` is unknown

### Search

```
Search(ctx context.Context, projectSlug string, query string, opts SearchOptions) (*SearchResult, error)
```

Unified search across content and glossary indexes. Parses the query via `ParseQuery`, resolves scope via `ResolveScope` if present, then queries both FTS5 tables.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug
- `query` -- raw search input (may include scope prefix, e.g. `"book-1:dragon"`)
- `opts` -- `SearchOptions` with `Types` filter, `Limit` (default 20), `Offset`

**Returns:** `*SearchResult` with ranked results, total count, facet counts, and optional message. Returns `"Scope Not Found"` message with empty results if the scope path does not match any content.

**Search paths:**
1. **FTS5 MATCH** (when `ParsedQuery.UseFTS5` is true) -- Uses `MATCH` with BM25 ranking and `snippet()` for highlighted excerpts with `<mark>` tags. Supports prefix wildcards (`drag*`), phrases (`"fire dragon"`), and boolean operators (`dragon NOT castle`).
2. **LIKE fallback** (when `ParsedQuery.UseFTS5` is false) -- For suffix (`*dragon`) and infix (`dr*gon`) wildcards. Reads all rows from the FTS5 table and filters in Go using `strings.Contains`. No snippets or ranking in this path.

### FindDocumentsContaining

```
FindDocumentsContaining(ctx context.Context, projectSlug string, terms []string) ([]string, error)
```

Returns content IDs whose `plain_text` contains any of the given terms. Builds an FTS5 MATCH expression with OR: `"term1" OR "term2" OR "term3"`. Each term is quoted and internal double quotes are escaped.

Used by the ProseMirror glossary matcher as a pre-filter to skip documents that cannot possibly match a glossary term.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug
- `terms` -- list of terms to search for (OR'd together)

**Returns:** list of content UUIDs, or empty slice if no terms provided

### ReindexAll

```
ReindexAll(ctx context.Context, projectSlug string, contentLoader ContentLoader, glossaryLoader GlossaryLoader) error
```

Drops all existing FTS5 index entries for a project and re-indexes everything from scratch. Clears both `content_fts` and `glossary_fts` tables, then iterates through all data provided by the loader callbacks.

Individual indexing failures are non-fatal (warned and skipped) to avoid a single corrupt entry from blocking the entire reindex.

**Parameters:**
- `ctx` -- request context
- `projectSlug` -- project slug
- `contentLoader` -- callback to load all content documents (may be nil)
- `glossaryLoader` -- callback to load all glossary entries (may be nil)

**Returns:** error if clearing tables or loading data fails

## Internal Helpers

| Name | Description |
| ---- | ----------- |
| `searchContent` | Queries `content_fts` via FTS5 MATCH or Go-side wildcard filtering, with scope ID filtering |
| `searchGlossary` | Queries `glossary_fts` via FTS5 MATCH or Go-side wildcard filtering, with scope post-filtering via `glossaryMatchesScope` |
| `countContentFTS5` | COUNT query against `content_fts` for pagination totals |
| `glossaryMatchesScope` | Checks if a glossary entry's `scope_targets` include the searched scope path using `scope.ItemAppliesToContent` |
| `getProjectID` | Extracts the project UUID from the first `contents` row (all rows share the same project) |
| `makePlaceholders` | Builds comma-separated `?` placeholders for IN clauses |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `scope` | `backend/internal/scope` | `ItemAppliesToContent` for glossary scope matching |
| `logging` | `backend/internal/logging` | Structured logging with `"search"` component |
| `database/sql` | stdlib | Raw SQL execution for FTS5 virtual tables |
| `context` | stdlib | Request context for cancellation/timeout |
| `fmt` | stdlib | Error wrapping |
| `strings` | stdlib | String manipulation for query building and wildcard matching |
| `sync` | stdlib | `sync.Once` for `StubProvider` warning (in `stub.go`) |

## Side Effects

- Reads and writes FTS5 virtual tables in the per-project SQLite database
- Opens and closes database connections via the injected `DBOpener` for each operation
- `StubProvider.Search` logs a warning on first invocation via `sync.Once`

## Notes

> [!WARNING] The LIKE fallback path reads ALL rows from the FTS5 table into memory and filters in Go. For large projects with thousands of content documents, suffix/infix wildcard queries (`*dragon`, `dr*gon`) may have noticeable latency. This is an inherent limitation -- suffix wildcards require full scans regardless of indexing strategy.

> [!WARNING] `glossary_fts.scope_targets` is currently empty for all entries until glossary-v2 adds the scope field. All glossary entries are treated as project-wide (always included in search results regardless of scope filter).
