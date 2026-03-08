---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/content-scope-filter/","title":"contentScopeFilter.ts","tags":["file","content-frontend","utility","scope"],"updated":"2026-03-05T06:28:04.418-07:00"}
---


# `contentScopeFilter.ts`

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/utils/contentScopeFilter.ts`

> [!NOTE] Generic content-scope filtering utilities for the frontend. Determines whether a scoped item (glossary term, entity, note, etc.) applies to a given content path in the content hierarchy. Both the frontend (this module) and backend (`scope` package) implement the same algorithm to ensure parity.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ScopedItem` | interface | Generic interface for any item that can be scoped to content |
| `filterByScope` | function | Filter items to only those whose scope includes the given content |
| `isContentWithinTarget` | function | Check if content path is within a scope target |
| `isAncestorPath` | function | Check if a path is a proper ancestor of another |
| `buildSlugPathString` | function | Join slug path array to string |
| `parseSlugPathString` | function | Split slug path string to array |
| `getSlugPathDisplayName` | function | Get last segment of a slug path |
| `getParentPath` | function | Get parent path from a slug path |
| `areSiblingPaths` | function | Check if two paths share the same parent |
| `getPathDepth` | function | Count segments in a slug path |

## `ScopedItem` Interface

| Field | Type | Description |
| ----- | ---- | ----------- |
| `scope` | `string` | Scope level: `'project'` applies everywhere |
| `scopeTargets` | `string[]?` | Specific content paths this item applies to |

## Scope Algorithm

The `filterByScope()` function implements a three-tier filter:

1. **Project scope** — If `item.scope === 'project'`, the item applies to all content
2. **No targets** — If `scopeTargets` is empty or undefined, the item applies to all content at that scope level
3. **Path matching** — Otherwise, checks if ANY scope target is a prefix of the content's slug path

### Path Prefix Matching

`isContentWithinTarget(contentSlugPath, targetSlugPath)`:

- Parses the target string into segments (e.g., `"series-1/book-1"` → `["series-1", "book-1"]`)
- Content slug path must be at least as deep as the target
- Each target segment must match the corresponding content segment
- Example: content `["series-1", "book-1", "chapter-2", "scene-1"]` is within target `"series-1/book-1"` → `true`

## Key Functions

### `filterByScope<T>(items, slugPath, projectScopeValue?)`

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `items` | `T[]` (extends ScopedItem) | yes | — | Items to filter |
| `slugPath` | `string[]` | yes | — | Current content's path |
| `projectScopeValue` | `string` | no | `'project'` | Scope value meaning "whole project" |

**Returns:** `T[]` — filtered items

### `isAncestorPath(ancestorPath, descendantPath)`

Checks if one path is a proper ancestor of another. Returns `false` for same path.

### `getParentPath(slugPath)`

Returns the parent path (everything except the last segment), or empty string if at root.

## Imports / Dependencies

None — pure utility functions with no external imports.

## Side Effects

None. Pure functions with no state or registrations.
