---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/glossary/overview/","title":"Glossary System","tags":["feature","glossary","core"],"updated":"2026-03-07T18:43:14.229-07:00"}
---


# Glossary System

> [!NOTE] A comprehensive terminology management system that stores, searches, and automatically links glossary terms within editor content via ProseMirror marks with hover tooltips.

## User-Facing Behavior

Writers manage a project-level glossary of terms through a sidebar panel. They can create entries with rich metadata (definitions, categories, aliases, relationships, scoping), search and filter the list, and view full entry details. When writing in the editor, glossary terms are automatically underlined with dotted marks — hovering shows a tooltip with the short definition, and clicking (when enabled) navigates to the entry. Terms can be added directly from the editor context menu by selecting text and choosing "Add to Glossary." Invented terms are automatically added to the spellcheck dictionary.

## Scope

- **License:** Core
- **Modules involved:**
  - [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Glossary Backend]] — HTTP handler, repository, term scan handler
  - [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend]] — Panel, components, store, ProseMirror plugin
- **API endpoints:** [[Palimpsest-Writing-Software/api/glossary/overview\|Glossary API]] (6 endpoints)
- **Data models:**
  - [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|data-models/GlossaryEntry]] — 37-field Ent schema
  - [[Palimpsest-Writing-Software/data-models/Glossary\|data-models/Glossary]] — Parent entity (1:1 with Project)
  - [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|data-models/GlossaryEntryDTO]] — Transfer objects and TypeScript types

## Architecture

### Entity Relationship

```
Project ──1:1──> Glossary ──1:N──> GlossaryEntry
```

Each project has exactly one `Glossary` entity (auto-created on first access). Each `Glossary` contains many `GlossaryEntry` records.

### Layer Overview

```
Frontend (Svelte 5)
  Components → Store → API Client ──HTTP──> Backend (Go)
  ProseMirror Plugin ←─ Events                Handler → Repository → Ent ORM → SQLite
                                               Handler → Job Manager → TermScanHandler
```

**Frontend path:** User interacts with `GlossaryPanel` (state-machine view router) which delegates to `GlossaryList`, `GlossaryEntryDetail`, or `GlossaryEntryForm`. All data flows through `glossaryStore` which calls `glossaryApi` HTTP client methods.

**Backend path:** `GlossaryHandler.Glossary()` dispatches by path + HTTP method to CRUD handler methods. Each calls the `ProjectStore` repository interface. The SQLite implementation opens the per-project database, performs the operation via Ent ORM, and manages reciprocal relationships, FTS5 indexing, and slug generation.

**Background path:** After create/update/delete, the handler submits a `term_scan` job. The `TermScanHandler` processes it asynchronously — scanning all content documents for the term, applying or removing `glossaryLink` ProseMirror marks, and updating usage counts.

**Editor path:** The `createGlossaryMarkPlugin()` ProseMirror plugin provides hover tooltips (cached, fetched from API) and click-to-navigate behavior. Events (`glossary:entry-saved`, `glossary:entry-deleted`) trigger cache clearing and content reloading.

### Key Design Decisions

- **Per-project SQLite databases** — each project has its own `.db` file; the glossary is accessed through the project's database
- **Lazy glossary creation** — the `Glossary` parent entity is auto-created on first access via `getOrCreateGlossary()`
- **Reciprocal relationships** — when entry A adds B to `broaderTerms`, B automatically gets A in `narrowerTerms` (6 relationship types, 4 symmetric)
- **Background term scanning** — mark application happens asynchronously via the job manager to avoid blocking CRUD responses
- **Graceful degradation** — search (FTS5), job submission, and indexing are all optional; operations proceed if any subsystem is nil
- **Post-query tag filtering** — SQLite JSON fields don't support efficient array-contains; tag matching happens in Go after the query
- **Self-reference prevention** — relationship arrays are stripped of the entry's own ID after save

## Dataflow Diagrams


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/create-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Create Entry: Full Flow

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/read-entries-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Read Entries: List, Get, and Occurrences

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/update-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Update Entry: With Reciprocal Diffs

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/delete-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Delete Entry: Cleanup and Mark Removal

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/term-scan-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Term Scan: Background Mark Processing

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/editor-integration-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Editor Integration: Tooltip, Click, Context Menu

</div>





# Excalidraw Data

## Text Elements




</div></div>


## Event System

Three glossary-specific events flow through `moduleEventBus`:

| Event | Payload | Dispatched By | Subscribers |
| ----- | ------- | ------------- | ----------- |
| [[Palimpsest-Writing-Software/api/glossary/event-glossary-view-term\|glossary:view-term]] | `{ id }` | TextEditor (mark click) | GlossaryPanel → `navigateToEntry()` |
| [[Palimpsest-Writing-Software/api/glossary/event-glossary-entry-saved\|glossary:entry-saved]] | `{ id }` | GlossaryEntryForm | TextEditor (clear cache), Editor page (reload 3s), Detail (reload usage 4s) |
| [[Palimpsest-Writing-Software/api/glossary/event-glossary-entry-deleted\|glossary:entry-deleted]] | `{ id }` | glossaryStore | TextEditor (clear cache), Editor page (reload 3s) |

Cross-module: `content:selected` used by GlossaryEntryDetail to navigate to content documents from occurrence results.

## Spellcheck Integration

After creating or updating an entry, the form calls `dictionaryStore.addWordIfMisspelled()` for the term and all aliases (2s delay). Invented words are added to the custom dictionary; real dictionary words are skipped. Uses retry logic with exponential backoff (3 attempts).

## Security

No authentication or authorization is enforced at the glossary level — all operations are scoped to the local project database. The per-project SQLite isolation means one project's glossary cannot access another's data. Input validation enforces non-empty term and shortDefinition on create, and at-least-one-field on update. Duplicate terms are rejected (case-insensitive). No user-supplied content is executed; ProseMirror marks use data attributes, not inline scripts.

> [!WARNING] The `image_url` field accepts arbitrary URLs. If this is ever rendered as `<img src>`, it could be an XSS vector. Currently it is only stored, not rendered in the UI.

## Performance

- **Tag filtering is O(n)** — runs in Go after the database query returns all matching entries. Mitigated by the 100-entry limit per page.
- **Reciprocal relationship sync is O(k)** per relationship type — for each added/removed ID, loads and saves the target entry. With 6 relationship types and small relationship arrays, this is typically <12 DB round-trips per save.
- **Term scan is O(n × m)** — scans n content documents for m terms/aliases per entry. Runs in a background job to avoid blocking the API response. `docChanged()` uses JSON serialization comparison to skip unchanged documents.
- **Tooltip caching** — `Map<string, GlossaryTermInfo>` in the ProseMirror plugin prevents redundant API calls. Cleared on entry save/delete events.
- **`cleanupRelationshipReferences()` on delete is O(all entries)** — queries every entry to remove the deleted ID from relationship arrays. Acceptable for glossaries under ~10K entries.

## 12-Factor Compliance

- **Config via environment:** Backend server port, storage path, and log level are configurable via environment variables.
- **Strict separation:** Frontend calls the backend exclusively via REST API; no direct database access from the UI.
- **Stateless processes:** Each API request is self-contained. The Go handler holds no request state between calls. The `TermScanHandler` uses callback functions to stay decoupled from the repository layer.
- **Dev/prod parity:** The same SQLite + Ent ORM stack runs in both development and production (Tauri sidecar). No database provider switching.

> [!TIP] The `TermScanHandler` callback pattern (entryLoader, contentLoader, bodySaver, countUpdater) is a good example of 12-Factor dependency injection — the handler depends on interfaces, not implementations.

## Logging

All components use injected loggers following project conventions:

| Layer | Component | Logger Name |
| ----- | --------- | ----------- |
| Backend | GlossaryHandler | `"glossary"` |
| Backend | TermScanHandler | `"term-scan"` |
| Backend | SQLite repository | via Store's injected logger |
| Frontend | glossaryStore | `'glossary'` |
| Frontend | GlossaryPanel | `'glossary-panel'` |
| Frontend | GlossaryList | `'glossary-list'` |
| Frontend | GlossaryEntryDetail | `'glossary-detail'` |
| Frontend | GlossaryEntryForm | `'glossary-form'` |
| Frontend | GlossaryTermPicker | `'term-picker'` |
| Frontend | ScopeTreePicker | `'scope-tree-picker'` |
| Frontend | CheckboxTree | `'checkbox-tree'` |

## Related

- [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Module: Glossary Backend]]
- [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Module: Glossary Frontend]]
- [[Palimpsest-Writing-Software/api/glossary/overview\|API: Glossary Service]]
- [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|Data Model: GlossaryEntry]]
- [[Palimpsest-Writing-Software/data-models/Glossary\|Data Model: Glossary]]
- [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|Data Model: GlossaryEntryDTO]]
