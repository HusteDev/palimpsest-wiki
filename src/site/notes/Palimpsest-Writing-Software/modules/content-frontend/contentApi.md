---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/content-api/","title":"content.ts (API)","tags":["file","content-frontend","api"],"updated":"2026-03-05T06:23:38.048-07:00"}
---


# `content.ts` (API)

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/api/content.ts`

> [!NOTE] Content API client providing typed HTTP methods for content CRUD, hierarchy queries, reordering, and utility functions for content type display. All methods delegate to the singleton [[Palimpsest-Writing-Software/modules/content-frontend/apiClient\|APIClient]].

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `getContentTypes` | function | Get all content types for a hierarchy |
| `getWritableContentType` | function | Get the content type with hasContent=true |
| `getContentHierarchy` | function | Build full ContentHierarchy from types |
| `listContent` | function | List content items (top-level or children) |
| `getContent` | function | Get single content by slug path |
| `createContent` | function | Create new content item |
| `updateContent` | function | Update content item by slug path |
| `deleteContent` | function | Delete content item (204 No Content) |
| `reorderContent` | function | Reorder items within parent (204 No Content) |
| `buildSlugPath` | function | Build slug path by walking parent chain |
| `getContentTypeLabel` | function | Get display label for content type |
| `getContentTypeIcon` | function | Get icon name for content type |
| `getContentTypeColor` | function | Get hex color for content type |
| `contentApi` | object | Grouped export of all functions |

## Key Functions

### `listContent(projectSlug, parentSlug?, options?)`

Lists content items. If `parentSlug` is provided, lists children of that parent; otherwise lists top-level items. Supports pagination, search, status filter, contentTypeId filter, and includeChildren flag.

**Endpoint:** `GET /api/projects/{slug}/content` or `GET /api/projects/{slug}/content/{parent}/children`

### `getContent(projectSlug, slugPath)`

Retrieves a single content item by its slug path array. Joins the array with `/` for the API path.

**Endpoint:** `GET /api/projects/{slug}/content/{slug-path...}`

### `createContent(projectSlug, parentSlug, input)`

Creates a new content item. Routes to the root endpoint or the parent's children endpoint based on `parentSlug`.

**Endpoint:** `POST /api/projects/{slug}/content` or `POST /api/projects/{slug}/content/{parent}/children`

### `buildSlugPath(content, allContent)`

Client-side utility that walks the parent chain by matching `parentId` against the `allContent` array. Prepends each parent's slug to build the full path from root.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `apiClient` | `./client` | Singleton HTTP client |
| Content types | `$lib/types/content` | TypeScript interfaces |
| `APIResponse` | `$lib/types/project` | Wrapper response type |

## Side Effects

None — pure API functions.
