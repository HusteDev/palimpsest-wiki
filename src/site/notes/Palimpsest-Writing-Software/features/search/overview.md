---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/search/overview/","title":"Search Engine","tags":["feature","search","core"],"updated":"2026-03-05T06:38:34.755-07:00"}
---


# Search Engine

> [!NOTE] A full-text search system powered by SQLite FTS5 that enables writers to find content and glossary entries across a project using ranked results with snippet highlighting, scoped queries, and live editor match decorations.

## User-Facing Behavior

Writers search their project using a search bar in the application header. Typing a query (minimum 2 characters, 300ms debounce) triggers a full-text search across all content documents and glossary entries. Results appear in a sidebar panel grouped by type (content vs glossary) with highlighted snippet excerpts. Clicking a content result navigates to that document in the editor; clicking a glossary result opens the glossary panel to that entry's detail view.

While search results are displayed, matching terms are highlighted directly in the active editor document using visual decorations (yellow background). These decorations are purely visual and never modify the document content.

**Query syntax** supports:
- Simple terms: `dragon` — matches any occurrence
- Multiple terms: `fire dragon` — implicit AND, matches both words
- Exact phrases: `"fire dragon"` — matches the exact sequence
- Prefix wildcards: `drag*` — matches "dragon", "dragonfly", etc. (FTS5 native)
- Suffix/infix wildcards: `*gon`, `dr*on` — LIKE fallback (slower)
- Boolean NOT: `dragon NOT castle` — excludes "castle" matches
- Scoped queries: `book-1:dragon` — searches only within content slug "book-1"
- Scoped paths: `series-1/book-1:dragon` — multi-level hierarchy scope
- Wildcard scopes: `book-*:dragon` — searches all slugs matching "book-*"

## Scope

- **License:** Core
- **Modules involved:**
  - [[Palimpsest-Writing-Software/modules/search-backend/overview\|Search Backend]] — FTS5 provider, query parser, scope resolver, HTTP handler, reindex job
  - [[Palimpsest-Writing-Software/modules/search-frontend/overview\|Search Frontend]] — SearchBar, SearchResults panel, search store, ProseMirror highlight plugin
- **API endpoints:** [[Palimpsest-Writing-Software/api/search/overview\|Search API]] (1 endpoint)
- **Data models:**
  - [[Palimpsest-Writing-Software/data-models/SearchResult\|data-models/SearchResult]] — Go structs and TypeScript interfaces for search requests/responses

## Architecture

### Index Architecture

```
Per-Project SQLite Database
  ├── content_fts (FTS5 virtual table)
  │     Columns: content_id (UNINDEXED), title, plain_text, tags, slug_path
  └── glossary_fts (FTS5 virtual table)
        Columns: entry_id (UNINDEXED), term, definition, aliases, tags, scope_targets
```

Each project has its own SQLite database with two FTS5 virtual tables. Content documents are indexed with plain text extracted from ProseMirror JSON. Glossary entries are indexed with their term, definition, aliases, tags, and scope targets.

### Layer Overview

```
Frontend (Svelte 5)
  SearchBar → searchStore → searchApi ──HTTP──> Backend (Go)
  SearchResults ← store                         SearchHandler → SearchProvider → FTS5
  ProseMirror Plugin ← store.query              QueryParser → ScopeResolver → SQLite

Incremental Indexing:
  Content CRUD ──> Repository ──> SearchProvider.IndexContent()
  Glossary CRUD ──> Repository ──> SearchProvider.IndexGlossaryEntry()

Background Jobs:
  JobManager ──> ReindexHandler ──> SearchProvider.ReindexAll()
```

**Frontend path:** User types in `SearchBar` (header toolbar), which debounces input and calls `searchStore.search()`. The store calls `searchApi.searchProject()` (HTTP GET), receives ranked results, and auto-opens the `SearchResults` panel. Simultaneously, `TextEditor` watches `$searchStore.query` via a reactive `$effect` and updates the `searchHighlight` ProseMirror plugin to render match decorations.

**Backend path:** `SearchHandler.Search()` validates query parameters, delegates to `SearchProvider.Search()`. The `FTS5Provider` parses the query via `QueryParser`, optionally resolves scope via `ScopeResolver`, executes FTS5 MATCH queries with BM25 ranking, generates highlighted snippets, and returns paginated results with facet counts.

**Indexing path:** Content and glossary CRUD operations in the SQLite repository call `SearchProvider.IndexContent()` or `SearchProvider.IndexGlossaryEntry()` after each mutation. These use a DELETE-then-INSERT upsert pattern to keep the FTS5 index current. On delete, `RemoveFromIndex()` removes the entry.

**Reindex path:** A `search_reindex` background job can be submitted to rebuild the entire index. The `ReindexHandler` uses callback functions (`ContentLoader`, `GlossaryLoader`) to load all indexable data, then calls `ReindexAll()` which drops and recreates the FTS5 tables.

### Provider Pattern

```
SearchProvider (interface)
  ├── FTS5Provider    — SQLite FTS5 (Core license, desktop)
  └── StubProvider    — No-op fallback (graceful degradation)
```

The `SearchProvider` interface abstracts all search operations. `FTS5Provider` is the production implementation. `StubProvider` is a no-op fallback that logs a single warning and returns empty results — used when the search subsystem is unavailable. This pattern is designed to accommodate a future PostgreSQL `tsvector` implementation for the Pro license.

### Key Design Decisions

- **Provider pattern** — backend-agnostic `SearchProvider` interface allows swapping FTS5 for PostgreSQL tsvector without changing callers
- **Per-project FTS5 virtual tables** — each project's SQLite database has its own `content_fts` and `glossary_fts` tables, created idempotently on first access via `EnsureIndexes()`
- **Incremental indexing** — every content and glossary CRUD operation immediately updates the search index (no batch reindex required for normal operations)
- **Graceful degradation** — if the search provider fails, the HTTP handler returns 200 OK with empty results and a message, never 500
- **Upsert via DELETE+INSERT** — FTS5 doesn't support UPDATE; the provider deletes then re-inserts on every index operation
- **Query parsing with scope resolution** — the colon `:` separator allows scoped searches; scope resolution uses recursive CTEs to walk the content tree
- **LIKE fallback for non-prefix wildcards** — FTS5 only supports prefix wildcards natively (`drag*`); suffix/infix wildcards (`*gon`, `dr*on`) fall back to SQL LIKE queries
- **Visual-only editor decorations** — search highlights use ProseMirror `Decoration.inline()`, never modifying the document model
- **Callback-based reindex** — `ReindexHandler` uses `ContentLoader` and `GlossaryLoader` function types to avoid circular imports with the repository package

## Dataflow Diagrams

- [[Palimpsest-Writing-Software/features/search/diagrams/search-query-flow\|Search Query: Full Lifecycle]]
- [[Palimpsest-Writing-Software/features/search/diagrams/indexing-content-flow\|Indexing: Content Create/Update and Delete]]
- [[Palimpsest-Writing-Software/features/search/diagrams/indexing-glossary-flow\|Indexing: Glossary Create/Update and Delete]]
- [[Palimpsest-Writing-Software/features/search/diagrams/indexing-reindex-flow\|Indexing: Full Reindex Job]]
- [[Palimpsest-Writing-Software/features/search/diagrams/scope-resolution-flow\|Scope Resolution: Path Parsing and Tree Walking]]
- [[Palimpsest-Writing-Software/features/search/diagrams/editor-highlight-flow\|Editor Highlight: ProseMirror Decorations]]

## Cross-Feature Integration

### Glossary Integration

Search results include glossary entries alongside content documents. When a user clicks a glossary result in the `SearchResults` panel, it dispatches a [[Palimpsest-Writing-Software/api/glossary/event-glossary-view-term\|glossary:view-term]] event, which the `GlossaryPanel` subscribes to and navigates to the entry detail view.

The `FindDocumentsContaining()` method on `SearchProvider` is also used by the glossary system's [[Palimpsest-Writing-Software/modules/glossary-backend/term_scan_handler.go\|TermScanHandler]] as a pre-filter to identify which content documents might contain a glossary term before performing the full mark scan.

### Content Navigation

When a user clicks a content result, the `SearchResults` component fetches the full content object via `contentApi.getContent()` and dispatches a `content:selected` event through the `moduleEventBus`, which the editor page subscribes to for loading the selected document.

## Security

No authentication or authorization is enforced at the search level — all operations are scoped to the local project database. The per-project SQLite isolation means one project's search index cannot query another's data. The query parser sanitizes user input for FTS5 safety (escaping special characters). No user-supplied content is executed; snippets use `<mark>` tags generated by FTS5's built-in `snippet()` function.

> [!WARNING] The `snippet` field in search results contains HTML (`<mark>` tags). The frontend renders this via `{@html}` in Svelte. The HTML is generated server-side by SQLite's `snippet()` function from indexed text, not from raw user input, so XSS risk is minimal. However, if the indexing pipeline ever stores unescaped HTML in FTS5 columns, this could become a vector.

## Performance

- **BM25 ranking** — FTS5 provides built-in BM25 relevance scoring; results are ordered by score (lower = more relevant)
- **Snippet extraction** — FTS5 `snippet()` generates highlighted excerpts server-side, avoiding transferring full document text
- **Default limit 20, max 100** — pagination prevents unbounded result sets
- **Debounce 300ms** — frontend debounces input to avoid excessive API calls during typing
- **Scope resolution uses recursive CTEs** — efficient tree walking in SQLite, but O(depth) for each scope segment
- **LIKE fallback is O(n)** — suffix/infix wildcards scan all indexed text; mitigated by being the exception path (most queries use FTS5 MATCH)
- **Decoration rebuild on doc change** — when the editor loads new content while a search is active, the plugin re-scans the full document; this is O(n) per document but only triggered on content swap, not on every keystroke

## 12-Factor Compliance

- **Config via environment:** Backend server port, storage path, and log level are configurable via environment variables.
- **Strict separation:** Frontend calls the backend exclusively via REST API; no direct database access from the UI.
- **Stateless processes:** Each search request is self-contained. The handler holds no request state between calls.
- **Backing service abstraction:** Search is treated as a backing service via the `SearchProvider` interface. The app degrades gracefully if the search subsystem is unavailable.
- **Dev/prod parity:** The same FTS5 + SQLite stack runs in both development and production (Tauri sidecar).

> [!TIP] The `SearchProvider` interface is the canonical example of 12-Factor backing service abstraction in this codebase. The `StubProvider` exists specifically to satisfy the principle that the app must function without any backing service being available.

## Logging

| Layer | Component | Logger Name |
| ----- | --------- | ----------- |
| Backend | SearchHandler | `"handler-search"` |
| Backend | FTS5Provider | `"search"` |
| Backend | QueryParser | `"search"` (package-level) |
| Backend | ScopeResolver | `"search"` (package-level) |
| Backend | ReindexHandler | `"search-reindex"` |
| Frontend | searchStore | `'search'` |
| Frontend | SearchBar | `'search-bar'` |
| Frontend | SearchResults | `'search-results'` |
| Frontend | searchHighlight plugin | `'search-highlight'` |

## Related

- [[Palimpsest-Writing-Software/modules/search-backend/overview\|Module: Search Backend]]
- [[Palimpsest-Writing-Software/modules/search-frontend/overview\|Module: Search Frontend]]
- [[Palimpsest-Writing-Software/api/search/overview\|API: Search Service]]
- [[Palimpsest-Writing-Software/data-models/SearchResult\|Data Model: SearchResult]]
- [[Palimpsest-Writing-Software/features/glossary/overview\|Feature: Glossary System]]
