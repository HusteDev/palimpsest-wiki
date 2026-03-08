---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-backend/overview/","title":"Content Backend Module","tags":["module","content-backend"],"updated":"2026-03-05T06:20:53.105-07:00"}
---


# Content Backend

> [!NOTE] Go HTTP handlers and SQLite repository implementation for hierarchical content CRUD, tree operations, reordering, word count aggregation, FTS5 search indexing, and glossary mark cleanup on deletion.

## Responsibilities

- Route and handle HTTP requests for content CRUD operations
- Provide content type listing by hierarchy name
- Manage content tree operations (recursive tree building, reorder)
- Calculate word counts from ProseMirror JSON and aggregate up the parent chain
- Index content in FTS5 for full-text search on create/update, remove on delete
- Clean up glossary entry counts when content containing glossaryLink marks is deleted
- Seed default "book-series" content type hierarchy on project creation
- Auto-title management with two-pass rename for UNIQUE constraint safety

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `Handler.Content` | method | Route dispatcher for `/api/projects/{slug}/content/...` |
| `Handler.ContentTypes` | method | Route dispatcher for `/api/projects/{slug}/content-types/...` |
| `Store.CreateContent` | method | Create content with slug generation, order calculation, FTS5 index |
| `Store.GetContent` | method | Retrieve content by slug path (tree navigation) |
| `Store.GetContentByID` | method | Retrieve content by UUID |
| `Store.ListContent` | method | Paginated content listing with filters |
| `Store.UpdateContent` | method | Update content with order shifting, word count recalc, FTS5 re-index |
| `Store.DeleteContent` | method | Delete with cascade, FTS5 removal, sibling reorder, parent word count |
| `Store.GetContentTree` | method | Recursive tree building with maxDepth |
| `Store.ReorderContent` | method | Transactional reorder within a parent |
| `Store.RecalculateWordCount` | method | Recursive word count aggregation up parent chain |
| `Store.ListAllContent` | method | Flat list of all content (for background jobs) |
| `Store.UpdateContentBody` | method | Body-only update with word count recalc (for background jobs) |
| `Store.ReorderAndRenameSiblings` | method | Two-pass reorder and auto-title rename |

## Internal Structure

**HTTP Handlers** — Two handler files route content-related requests:
- `[[modules/content-backend/contentHandler|contentHandler]]` — Content CRUD, tree, and reorder endpoints
- `[[modules/content-backend/contentTypesHandler|contentTypesHandler]]` — Content type listing and retrieval

**Repository** — The SQLite implementation handles all persistence:
- `[[modules/content-backend/contentRepo|contentRepo]]` — CRUD operations, slug path navigation, FTS5 indexing, auto-title management
- `[[modules/content-backend/treeRepo|treeRepo]]` — Tree building, transactional reorder, recursive word count aggregation
- `[[modules/content-backend/contentTypesSeed|contentTypesSeed]]` — Default "book-series" hierarchy seeding

**Interfaces and DTOs** — Defined in the `repository` package:
- `ProjectStore` interface — Contract for all data persistence operations
- `ContentDTO`, `ContentTypeDTO`, `ContentTreeDTO` — Transfer objects
- `CreateContentInput`, `UpdateContentInput`, `ListContentInput` — Input DTOs

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `ent` (Ent ORM) | external | Database query builder and schema management |
| `repository` package | internal | DTOs and `ProjectStore` interface |
| `prosemirror` package | internal | Mark extraction for glossary cleanup on delete |
| `domain/project` | internal | `Status` enum for content status field |
| `http/request` | internal | Query parameter parsing, body validation |
| `http/response` | internal | Standardized JSON responses |
| `http/contextx` | internal | Project slug extraction from request context |
| `utils` | internal | `Slugify()` for slug generation |
| `logging` | internal | Injected logger with `[content]` component |

## Logging

Uses injected logger (`s.log` on Store, `h.log` on Handler) with bracketed prefixes:
- `[UpdateContent]` — Order change detection and sibling shifting
- `[DeleteContent]` — Glossary count decrement failures
- `[ReorderAndRenameSiblings]` — Two-pass rename progress
- `[buildSlugPath]` — Slug path resolution
- `[ListAllContent]` — Bulk content loading for background jobs
- `[UpdateContentBody]` — Body-only updates for background jobs

## Related

- [[Palimpsest-Writing-Software/features/content-management/overview\|Feature: Content Management]]
- [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend Module]]
- [[Palimpsest-Writing-Software/data-models/Content\|Data Model: Content]]
- [[Palimpsest-Writing-Software/data-models/ContentType\|Data Model: ContentType]]
- [[Palimpsest-Writing-Software/api/content/overview\|Content API Service]]
