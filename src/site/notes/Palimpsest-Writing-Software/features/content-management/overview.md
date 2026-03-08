---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/content-management/overview/","title":"Content Management","tags":["feature","content-management","core"],"updated":"2026-03-05T06:20:10.471-07:00"}
---


# Content Management

> [!NOTE] Hierarchical content system that lets authors organize writing projects into nested structures (Series > Book > Chapter > Scene), with three-tier persistence, word count aggregation, auto-titling, and full-text search indexing.

## Overview

Content Management is the central feature of Palimpsest. It provides:

- **Hierarchical content structure** — Content is organized into a tree using configurable ContentType levels. The default "book-series" hierarchy is Series > Book > Chapter > Scene
- **Leaf-only writing** — Only the deepest level (e.g., Scene) stores ProseMirror JSON content; parent levels are structural containers with aggregated word counts
- **Three-tier persistence** — Editor changes flow through localStorage drafts (3s debounce) → autosave service (configurable timer + navigation triggers) → backend SQLite via REST API
- **Auto-titling** — Content items can be auto-titled based on their type and position (e.g., "Scene-1", "Scene-2"), with a two-pass rename to avoid UNIQUE constraint collisions
- **Word count aggregation** — Leaf content calculates word count directly from ProseMirror JSON; parent types aggregate children's counts recursively up the tree
- **Full-text search indexing** — Content is indexed into FTS5 on every create/update and removed on delete, enabling project-wide search
- **Glossary mark cleanup** — When content is deleted, glossary link marks are extracted and their usage/document counts are decremented

## Architecture

### Content Hierarchy

The content system uses a two-entity model:

1. **[[Palimpsest-Writing-Software/data-models/ContentType\|ContentType]]** — Defines the structural levels of a hierarchy. Each level has a `name`, `level` number, and a `hasContent` flag. Only the level with `hasContent=true` stores ProseMirror content
2. **[[Palimpsest-Writing-Software/data-models/Content\|Content]]** — Individual items in the tree. Each references a ContentType, a project, and optionally a parent Content

The default "book-series" hierarchy seeds four levels on project creation:

| Level | Name    | hasContent | Icon        | Purpose                   |
| ----- | ------- | ---------- | ----------- | ------------------------- |
| 1     | Series  | false      | `library`   | Top-level grouping        |
| 2     | Book    | false      | `book`      | Individual books           |
| 3     | Chapter | false      | `file-text` | Chapter containers         |
| 4     | Scene   | true       | `square-pen`| Actual writing content     |

### Three-Tier Persistence

Content flows through three persistence layers:

1. **[[Palimpsest-Writing-Software/modules/content-frontend/localDraftService\|LocalDraftService]]** (localStorage) — Debounced 3-second save of every editor change. Provides crash recovery and offline resilience. Data stored as `DraftData` (html, text, json, wordCount, charCount, lastModified)

2. **[[Palimpsest-Writing-Software/modules/content-frontend/saveService\|SaveService]]** (backend sync) — Flushes localStorage draft, then sends to backend via `contentApi.updateContent()`. Recalculates word count from ProseMirror JSON. Tracks unsynced documents in localStorage for crash recovery

3. **[[Palimpsest-Writing-Software/modules/content-frontend/autosaveService\|AutosaveService]]** (trigger layer) — Two trigger modes:
   - **Time-based**: PollingTimer at configurable interval (user preference, in minutes)
   - **Navigation-based**: Three modes via `saveOnNavigation` preference: `enable` (save silently), `disable` (keep draft), `prompt` (show UnsavedChangesModal)

### Slug Path Addressing

Content is addressed via slug paths — arrays of slugs from root to leaf. For example, a scene might be addressed as `["my-series", "book-1", "chapter-3", "scene-2"]`. The backend navigates the tree by walking from the root slug downward, matching each segment against children of the current node.

Slug paths are used for:
- REST API endpoint routing (`/api/projects/{slug}/content/{slug-path...}`)
- Frontend navigation (`/editor/{slug-path}`)
- Scope filtering (determining which glossary terms apply to which content)

### Auto-Titling System

Content items with `isAutoTitled=true` receive sequential titles based on their ContentType name and position: `Scene-1`, `Scene-2`, etc. When items are reordered or deleted, the `ReorderAndRenameSiblings` method uses a **two-pass approach** to avoid UNIQUE constraint violations:

1. **Pass 1**: Set all auto-titled items to temporary slugs (`__temp_reorder_{id}`)
2. **Pass 2**: Set final titles and slugs (`Scene-1`, `scene-1`)

### Word Count Aggregation

Word counting follows two strategies:

- **Leaf content** (`hasContent=true`): Word count is calculated by `countWordsFromProseMirror()`, which recursively extracts text nodes and splits on whitespace
- **Parent content** (`hasContent=false`): Word count is the sum of all direct children's word counts, calculated by `RecalculateWordCount()` which walks up the tree recursively

The frontend uses `wordCountService` for reactive word count display in the content tree.

## Frontend Architecture

The frontend manages content through several coordinated layers:

- **[[Palimpsest-Writing-Software/modules/content-frontend/contentTypes\|Content Types]]** (`src/lib/types/content.ts`) — TypeScript interfaces for Content, ContentType, CreateContentInput, UpdateContentInput, and related DTOs
- **[[Palimpsest-Writing-Software/modules/content-frontend/contentApi\|Content API]]** (`src/lib/api/content.ts`) — HTTP client wrapper with CRUD methods plus utility functions for labels, icons, colors
- **[[Palimpsest-Writing-Software/modules/content-frontend/contentScopeFilter\|Content Scope Filter]]** (`src/lib/utils/contentScopeFilter.ts`) — Generic scope-matching utilities for determining which scoped items (glossary terms, etc.) apply to a given content path

### Save Service Stack

| Layer | Service | Storage | Trigger | Latency |
| ----- | ------- | ------- | ------- | ------- |
| 1 | [[Palimpsest-Writing-Software/modules/content-frontend/localDraftService\|LocalDraftService]] | localStorage | Every editor change | 3s debounce |
| 2 | [[Palimpsest-Writing-Software/modules/content-frontend/saveService\|SaveService]] | Backend SQLite | Called by layer 3 | Immediate |
| 3 | [[Palimpsest-Writing-Software/modules/content-frontend/autosaveService\|AutosaveService]] | (orchestrator) | Timer or navigation | Configurable |

## Backend Architecture

The backend provides REST endpoints for content CRUD, tree operations, and reordering:

- **[[Palimpsest-Writing-Software/modules/content-backend/contentHandler\|Content Handler]]** (`handlers/content.go`) — HTTP handler with path-based routing for content operations
- **[[Palimpsest-Writing-Software/modules/content-backend/contentTypesHandler\|Content Types Handler]]** (`handlers/content_types.go`) — HTTP handler for content type listing
- **[[Palimpsest-Writing-Software/modules/content-backend/contentRepo\|Content Repository]]** (`sqlite/content.go`) — Ent-based CRUD with slug generation, FTS5 indexing, auto-title management
- **[[Palimpsest-Writing-Software/modules/content-backend/treeRepo\|Tree Repository]]** (`sqlite/tree.go`) — Tree operations: recursive tree building, transactional reorder, word count aggregation

### Data Flow: Content Update

1. Editor `onUpdate` fires with `{ html, text, json }`
2. `localDraftService.scheduleSave()` starts 3-second debounce timer
3. Timer fires → `performSave()` writes to localStorage
4. Autosave timer fires → `saveService.saveContent()`:
   a. Flushes pending localStorage save
   b. Reads draft from localStorage
   c. Recalculates word count from ProseMirror JSON
   d. Calls `contentApi.updateContent()` → REST API
5. Backend `UpdateContent` handler:
   a. Parses slug path from URL
   b. Applies field updates to Ent entity
   c. If content changed: recalculates word count, walks up parent chain
   d. Re-indexes content in FTS5 search
   e. Returns updated ContentResponse

## Integration Points

### Glossary System

- On content **delete**: Extracts `glossaryLink` marks from ProseMirror JSON via `prosemirror.ExtractMarksOfType()`, then decrements `usageCount` and `docCount` on affected glossary entries
- **Scope filtering**: Glossary terms use slug paths as scope targets; `contentScopeFilter.filterByScope()` determines which terms apply to the current content

### Search Engine

- On content **create/update**: `indexContentForSearch()` extracts plain text from ProseMirror JSON and indexes title, plain_text, tags, slug_path into FTS5
- On content **delete**: `removeContentFromSearch()` removes the document from the FTS5 index

### ProseMirror Editor

- The editor page (`/editor/[slug]`) loads content via slug path, initializes the ProseMirror editor with the stored JSON, and connects the save service stack
- Draft recovery: On mount, checks if a localStorage draft is newer than the backend content and offers to recover it

## Dataflow Diagrams

- [[Palimpsest-Writing-Software/features/content-management/diagrams/create-content-flow\|Create Content: Full Lifecycle]]
- [[Palimpsest-Writing-Software/features/content-management/diagrams/save-content-flow\|Save Content: Three-Tier Persistence]]
- [[Palimpsest-Writing-Software/features/content-management/diagrams/delete-content-flow\|Delete Content: Cleanup Chain]]

## Design Decisions

1. **Leaf-only content storage** — Only the deepest ContentType level stores ProseMirror JSON. Parent levels are structural containers. This simplifies content editing (one editor per Scene) while allowing complex organizational structures
2. **Three-tier persistence** — localStorage provides instant feedback and crash recovery; autosave handles backend sync. This eliminates the "save button" UX pattern and prevents data loss
3. **Two-pass rename** — Auto-titled items use temporary slugs during reorder to avoid UNIQUE constraint failures in SQLite. This is more complex but prevents edge cases where swapping positions would violate the slug+parent uniqueness constraint
4. **Slug path addressing** — Using slug arrays instead of UUIDs for routing provides human-readable URLs and enables the hierarchical scope system

## Security

- Content is stored in per-project SQLite databases, providing natural isolation between projects
- All content operations require a valid project slug resolved from the request context
- The `X-Project-Type: local` header is set on all API requests (Alpha: all projects are local)
- Content body parsing uses `request.ParseSafely()` which limits request body size

## Performance

- **Debounced localStorage saves** (3s) prevent excessive writes during rapid typing
- **Word count recalculation** walks up the parent chain — O(depth) per save, where depth is typically 3-4 levels
- **FTS5 indexing** uses delete+insert pattern which is efficient for SQLite's full-text search
- **Ent query builder** uses indexed fields (slug+parent, order+parent) for efficient lookups
- **Tree building** uses recursive queries with maxDepth limiting to prevent unbounded recursion

## Related

- [[Palimpsest-Writing-Software/features/editor/overview\|ProseMirror Editor]]
- [[Palimpsest-Writing-Software/features/glossary/overview\|Glossary System]]
- [[Palimpsest-Writing-Software/features/search/overview\|Search Engine]]
- [[Palimpsest-Writing-Software/_index/moc-core-features\|MOC: Core Features]]
