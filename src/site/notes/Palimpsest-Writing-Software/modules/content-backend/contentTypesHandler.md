---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-backend/content-types-handler/","title":"content_types.go","tags":["file","content-backend","handler"],"updated":"2026-03-05T06:24:52.427-07:00"}
---


# `content_types.go`

**Module:** [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend]]
**Path:** `backend/internal/http/handlers/content_types.go`

> [!NOTE] HTTP handler for content type listing and retrieval. Content types define the structural levels of a content hierarchy (e.g., Series -> Book -> Chapter -> Scene).

## Exports (Handler Methods)

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ContentTypes` | method | Route dispatcher for `/api/projects/{slug}/content-types/...` |
| `ListContentTypes` | method | `GET` — List types by hierarchy name |
| `GetContentType` | method | `GET` — Get single type by ID |

## Route Dispatch

| URL Pattern | Method | Handler |
| ----------- | ------ | ------- |
| `/api/projects/{slug}/content-types` | GET | `ListContentTypes` |
| `/api/projects/{slug}/content-types/{id}` | GET | `GetContentType` |

## Query Parameters

### `ListContentTypes`

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `hierarchy` | `string` | `"book-series"` | Hierarchy name to filter by |

## Response DTO

### `ContentTypeResponse`

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | UUID |
| `name` | `string` | Display name (e.g., "Chapter") |
| `hierarchyName` | `string` | Hierarchy this type belongs to |
| `level` | `int` | Level in hierarchy (1 = top) |
| `hasContent` | `bool` | Whether this level has ProseMirror content |
| `isSystem` | `bool` | Built-in vs user-created |
| `parentId` | `*string` | Parent type UUID |
| `metadata` | `map[string]any` | UI metadata (icon, color, labels) |
| `createdAt` | `string` | ISO 8601 timestamp |
| `updatedAt` | `string` | ISO 8601 timestamp |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `repository` | internal | `ContentTypeDTO` |
| `request` | internal | `QueryString` |
| `response` | internal | OK, NotFound, MethodNotAllowed |
| `contextx` | internal | `GetProjectSlugFromRequest` |

## Side Effects

None.
