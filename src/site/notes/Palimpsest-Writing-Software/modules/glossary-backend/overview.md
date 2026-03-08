---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-backend/overview/","title":"Glossary Backend Module","tags":["module","glossary","backend"],"updated":"2026-03-07T18:44:45.661-07:00"}
---


# Glossary Backend Module

> [!NOTE] Go backend module handling glossary CRUD operations, SQLite persistence via Ent ORM, reciprocal relationship management, FTS5 search indexing, and background term scanning.

## Responsibilities

- HTTP routing and request validation for 6 glossary API endpoints
- SQLite repository implementation (CRUD, slug generation, index text computation)
- Reciprocal relationship synchronization across 6 relationship types
- FTS5 search indexing and removal
- Background term scan job processing (apply/remove ProseMirror marks)
- Usage count tracking (set by term scan, decremented on content deletion)

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| [[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler]] | struct | HTTP handler with route dispatcher |
| [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store (glossary methods)]] | methods | SQLite repository CRUD implementation |
| [[Palimpsest-Writing-Software/modules/glossary-backend/term_scan_handler.go\|TermScanHandler]] | struct | Background job handler for term scanning |

## Internal Structure

The module is organized across three source files in different Go packages, unified by the module's shared purpose:

- [[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|handlers/glossary.go]] -- HTTP handler layer. Dispatches by URL path and HTTP method, validates request bodies, calls the repository store, and submits background term scan jobs. Lives in `backend/internal/http/handlers/`.
- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|sqlite/glossary.go]] -- SQLite repository layer. Implements CRUD via Ent ORM, manages slug generation, reciprocal relationships, FTS5 indexing, and DTO conversion. Lives in `backend/internal/repository/sqlite/`.
- [[Palimpsest-Writing-Software/modules/glossary-backend/term_scan_handler.go\|glossary/term_scan_handler.go]] -- Background job handler. Applies or removes `glossaryLink` ProseMirror marks in content documents. Decoupled from the repository via callback functions. Lives in `backend/internal/glossary/`.

Route registration happens in `backend/internal/http/server.go`, where `GlossaryHandler.Glossary` is mounted on the `/api/projects/{slug}/glossary` path prefix. Handler construction and callback wiring occur in `backend/cmd/server/main.go` (lines 159-206).

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|data-models/GlossaryEntry]] | internal | Ent ORM schema for glossary entries (37 fields) |
| [[Palimpsest-Writing-Software/data-models/Glossary\|data-models/Glossary]] | internal | Parent entity schema (1:1 with Project) |
| `repository.ProjectStore` | internal | Interface for store operations (CRUD, listing) |
| `search.SearchProvider` | internal | FTS5 indexing (optional, may be nil) |
| `jobs.JobSubmitter` | internal | Background job submission (optional, may be nil) |
| `prosemirror` package | internal | Mark manipulation for term scanning |
| `scope` package | internal | Content scope filtering (`ItemAppliesToContent`) |

## Dataflow Diagrams


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/create-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Create Entry Flow

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/update-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Update Entry Flow

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/delete-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Delete Entry Flow

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/term-scan-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Term Scan Flow

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Security

No authentication or authorization is enforced at the glossary level -- all operations are scoped to the local per-project SQLite database file. The per-project isolation means one project's glossary cannot access another's data.

**Input validation:**
- Create: non-empty `term` and `shortDefinition` required
- Update: at least one field must be non-nil
- Duplicate terms are rejected case-insensitively via `TermEqualFold` check

> [!WARNING] The `image_url` field accepts arbitrary URLs. If this is ever rendered as `<img src>`, it could be an XSS vector. Currently it is only stored, not rendered in the UI.

## Performance

- **Tag filtering is O(n) post-query** -- SQLite JSON fields do not support efficient array-contains queries. Tag matching runs in Go after the database query returns results. Mitigated by the 100-entry limit per page.
- **Reciprocal sync is O(k) per relationship type** -- for each added or removed ID, loads and saves the target entry. With 6 relationship types and small relationship arrays, this is typically fewer than 12 DB round-trips per save.
- **Term scan is O(n * m)** -- scans n content documents for m terms/aliases per entry. Runs in a background job to avoid blocking the API response. `docChanged()` uses JSON serialization comparison to skip unchanged documents.
- **`cleanupRelationshipReferences()` on delete is O(all entries)** -- queries every entry in the glossary to remove the deleted ID from relationship arrays. Acceptable for glossaries under ~10K entries.

## Related

- [[Palimpsest-Writing-Software/features/glossary/overview\|Feature: Glossary System]]
- [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Module: Glossary Frontend]]
- [[Palimpsest-Writing-Software/api/glossary/overview\|API: Glossary Service]]
- [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|Data Model: GlossaryEntry]]
- [[Palimpsest-Writing-Software/data-models/Glossary\|Data Model: Glossary]]
