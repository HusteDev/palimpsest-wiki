---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/content/overview/","title":"Content API Service","tags":["api","content"],"updated":"2026-03-05T06:49:36.291-07:00"}
---


# Content API Service

> [!NOTE] REST API service for hierarchical content CRUD, tree operations, and reordering. All endpoints are project-scoped under `/api/projects/{projectSlug}/`.

## Base URL

`/api/projects/{projectSlug}`

## Authentication

None (Alpha: local desktop app only). All requests include `X-Project-Type: local` header.

## Content Endpoints

### `GET /content`

List top-level content items (no parent).

**Query Parameters:**

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `limit` | int | 50 | Max items (1-100) |
| `offset` | int | 0 | Pagination offset |
| `search` | string | — | Filter by title/description |
| `status` | string | — | Filter by status |
| `contentTypeId` | string | — | Filter by content type |
| `includeChildren` | bool | false | Include direct children |

**Response:** `{ success: true, data: { content: ContentResponse[], total: int } }`

### `POST /content`

Create top-level content item.

**Body:** `CreateContentRequest` (title required, contentTypeId required)

**Response:** `201 Created` with `ContentResponse`

### `GET /content/{slug-path...}`

Get content item by full slug path.

**Response:** `ContentResponse`

### `PUT /content/{slug-path...}`

Update content item. All fields optional (partial update via pointer types).

**Body:** `UpdateContentRequest`

**Response:** `ContentResponse`

### `DELETE /content/{slug-path...}`

Delete content item and all children (cascade). Performs glossary mark cleanup before deletion.

**Response:** `204 No Content`

### `GET /content/{parent-slug}/children`

List children of a parent content item.

**Response:** Same as `GET /content`

### `POST /content/{parent-slug}/children`

Create content item as child of parent.

**Response:** `201 Created` with `ContentResponse`

### `PUT /content/{parent-slug}/reorder`

Reorder items within a parent.

**Body:** `{ items: [{ id: string, order: int }] }`

**Response:** `204 No Content`

### `GET /content/{slug}/tree`

Get recursive content tree from a root.

**Query Parameters:**

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `maxDepth` | int | 10 | Maximum tree depth |

**Response:** `ContentTreeResponse`

## Content Type Endpoints

### `GET /content-types`

List content types for a hierarchy.

**Query Parameters:**

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `hierarchy` | string | `"book-series"` | Hierarchy name |

**Response:** `{ success: true, data: { contentTypes: ContentTypeResponse[] } }`

### `GET /content-types/{id}`

Get single content type by ID.

**Response:** `ContentTypeResponse`

## Error Responses

| Status | Condition |
| ------ | --------- |
| `400 Bad Request` | Missing required fields, invalid JSON, wrong Content-Type |
| `404 Not Found` | Content or content type not found |
| `405 Method Not Allowed` | Wrong HTTP method for endpoint |
| `500 Internal Server Error` | Database or server error |

All errors return `{ success: false, error: string }`.

## Handler Implementation

- Content routes: [[Palimpsest-Writing-Software/modules/content-backend/contentHandler\|content.go]]
- Content type routes: [[Palimpsest-Writing-Software/modules/content-backend/contentTypesHandler\|content_types.go]]

## Related

- [[Palimpsest-Writing-Software/modules/content-frontend/contentApi\|Frontend API Client]]
- [[Palimpsest-Writing-Software/features/content-management/overview\|Content Management Feature]]
- [[Palimpsest-Writing-Software/data-models/Content\|Content Data Model]]
- [[Palimpsest-Writing-Software/data-models/ContentType\|ContentType Data Model]]
