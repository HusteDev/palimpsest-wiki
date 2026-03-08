---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-backend/tree-repo/","title":"tree.go","tags":["file","content-backend","repository","sqlite"],"updated":"2026-03-05T06:25:15.432-07:00"}
---


# `tree.go`

**Module:** [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend]]
**Path:** `backend/internal/repository/sqlite/tree.go`

> [!NOTE] Tree operations for content hierarchy: recursive tree building with depth limiting, transactional reorder, and recursive word count aggregation up the parent chain.

## Exports (Store Methods)

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `GetContentTree` | method | Build content tree from root slug (or forest of top-level items) |
| `ReorderContent` | method | Transactional batch reorder within a parent |
| `RecalculateWordCount` | method | Recursive word count aggregation |

## GetContentTree

Builds a recursive tree structure:

1. If `rootSlug` is provided: finds root content by slug path, loads with ContentType, then recursively builds tree
2. If `rootSlug` is nil: queries all top-level content (no parent), builds tree for each, wraps in a virtual root node

The `buildContentTree()` helper recursively queries children ordered by `order` field, with depth limiting via `maxDepth` parameter.

**Query parameters:**

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `rootSlug` | `*string` | nil | Root content slug (nil = full forest) |
| `maxDepth` | `int` | 10 | Maximum recursion depth (0 = unlimited) |

## ReorderContent

Transactional batch reorder:

1. Opens a database transaction
2. Updates each item's order field by ID
3. Commits transaction (rolls back on any error)

> [!IMPORTANT] This method updates order fields only. It does NOT handle auto-title renaming. Call `ReorderAndRenameSiblings()` (in content.go) for full reorder with rename support.

## RecalculateWordCount

Recursive word count aggregation:

1. Gets the content item with its ContentType edge
2. If `hasContent=true`: calculates word count from ProseMirror JSON using `countWordsFromProseMirror()`
3. If `hasContent=false`: sums `wordCount` from all direct children
4. Updates the content's word count
5. Recursively calls itself on the parent (if exists)

This ensures word counts are accurate at every level of the hierarchy after content updates.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `ent` | generated | Ent client, ordering |
| `content`, `project` | generated | Ent predicates |
| `repository` | internal | `ContentTreeDTO`, `ReorderItem` |

## Side Effects

- Database reads and writes via Ent
- `ReorderContent` uses database transactions for atomicity
