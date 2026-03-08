---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-backend/content-repo/","title":"content.go (Repository)","tags":["file","content-backend","repository","sqlite"],"updated":"2026-03-05T06:25:07.432-07:00"}
---


# `content.go` (Repository)

**Module:** [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend]]
**Path:** `backend/internal/repository/sqlite/content.go`

> [!NOTE] SQLite repository implementation for content CRUD operations, content type queries, slug path navigation, FTS5 search indexing, auto-title management, and bulk content operations for background jobs. All data is loaded from per-project SQLite database files.

## Exports (Store Methods)

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `GetContentTypesByHierarchy` | method | List content types by hierarchy name, ordered by level |
| `GetContentType` | method | Get single content type by ID |
| `CreateContentType` | method | Create a new content type |
| `CreateContent` | method | Create content with slug gen, order calc, FTS5 index |
| `GetContent` | method | Get content by slug path (tree navigation) |
| `GetContentByID` | method | Get content by UUID |
| `ListContent` | method | Paginated list with filters, parent scope, includeChildren |
| `UpdateContent` | method | Update with order shifting, word count recalc, FTS5 re-index |
| `DeleteContent` | method | Delete with cascade, FTS5 removal, sibling reorder |
| `ListAllContent` | method | Flat list of all content for background jobs |
| `UpdateContentBody` | method | Body-only update for background jobs |
| `ReorderAndRenameSiblings` | method | Two-pass reorder and auto-title rename |

## CreateContent Flow

1. Open project database, get project and content type entities
2. Resolve parent content (if `parentSlug` provided) via `getContentBySlug()`
3. Generate slug from title via `utils.Slugify()` if not provided
4. Calculate order: query max order among siblings, add 1
5. Build Ent `Create` query with all fields
6. Save and reload with edges (ContentType, Parent, Children)
7. Build slug path by walking up parent chain
8. Index content for FTS5 search (non-blocking, graceful degradation)

## UpdateContent Flow

1. Resolve content by slug path
2. Apply non-nil fields to Ent `Update` query
3. If content JSON changed: recalculate word count via `countWordsFromProseMirror()`
4. If order changed: shift siblings (up or down depending on direction), then call `ReorderAndRenameSiblings()`
5. If content changed: call `RecalculateWordCount()` to update parent chain
6. Reload with edges, build slug path
7. Re-index in FTS5 search

## DeleteContent Flow

1. Resolve content by slug path (single slug or multi-segment)
2. Capture parent ID for post-delete operations
3. Delete via Ent (cascade handles children)
4. Remove from FTS5 search index
5. Recalculate parent word count
6. Reorder remaining siblings and rename auto-titled items

## ReorderAndRenameSiblings (Two-Pass)

Handles reorder after delete or drag-and-drop:

**Pass 1**: Update order values to sequential (0, 1, 2...) and set auto-titled items to temporary slugs (`__temp_reorder_{id}`) to avoid UNIQUE constraint failures.

**Pass 2**: Set final titles and slugs for auto-titled items (`Scene-1`, `scene-1`).

## Slug Path Navigation

Two approaches for finding content by slug:

- `getContentBySlugPath(projectID, slugPath)` — Navigates from root slug downward, matching each segment against children
- `getContentBySlug(projectID, slug)` — Direct lookup by slug within project (slugs are unique per project)

## FTS5 Integration

| Method | Behavior |
| ------ | -------- |
| `indexContentForSearch` | Extracts plain text from PM JSON, indexes title + text + tags + slugPath |
| `removeContentFromSearch` | Removes document from FTS5 by content ID |

Both are no-ops if `s.search` (SearchProvider) is nil. Failures are logged but don't block the operation.

## Word Count

`countWordsFromProseMirror(doc)` recursively extracts text from ProseMirror JSON nodes, then splits on whitespace to count words. Used during create, update, and body-only update operations.

## Internal Helpers

| Function | Description |
| -------- | ----------- |
| `getContentBySlugPath` | Navigate tree by slug path segments |
| `getContentBySlug` | Direct slug lookup within project |
| `generateContentSlug` | Slugify title via `utils.Slugify()` |
| `nextSiblingOrder` | Get max order + 1 among siblings |
| `contentToDTO` | Convert Ent Content to repository ContentDTO |
| `contentTypeToDTO` | Convert Ent ContentType to repository ContentTypeDTO |
| `setSlugPathRecursive` | Set slug paths on DTO and all nested children |
| `buildSlugPath` | Walk up parent chain to build full slug path |
| `extractTextFromProseMirror` | Recursively extract text from PM JSON nodes |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `ent` | generated | Ent ORM client, predicates, queries |
| `content`, `contenttype`, `project` | generated | Ent predicate packages |
| `repository` | internal | DTOs and input types |
| `utils` | internal | `Slugify()` |

## Side Effects

- Opens and closes per-project SQLite database connections per operation
- Writes to FTS5 search index (via SearchProvider)
- Writes to Ent schema tables (content, content_type)
