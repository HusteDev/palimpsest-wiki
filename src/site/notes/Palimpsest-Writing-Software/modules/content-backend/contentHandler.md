---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-backend/content-handler/","title":"content.go (Handler)","tags":["file","content-backend","handler"],"updated":"2026-03-07T18:15:29.143-07:00"}
---


# `content.go` (Handler)

**Module:** [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend]]
**Path:** `backend/internal/http/handlers/content.go`

> [!NOTE] HTTP handler for content CRUD operations, tree queries, and reordering. Routes requests based on URL path structure and delegates to the ProjectStore interface. Includes glossary mark cleanup on content deletion.

## Exports (Handler Methods)

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Content` | method | Main route dispatcher for `/api/projects/{slug}/content/...` |
| `ListContent` | method | `GET` — Paginated content listing with filters |
| `CreateContent` | method | `POST` — Create content item |
| `GetContent` | method | `GET` — Retrieve content by slug path |
| `UpdateContent` | method | `PUT` — Update content item |
| `DeleteContent` | method | `DELETE` — Delete content with glossary cleanup |
| `ReorderContent` | method | `PUT` — Reorder items within parent |
| `GetContentTree` | method | `GET` — Recursive tree from root |

## Route Dispatch

The `Content()` method parses the URL path and routes to internal handlers:

| URL Pattern | Method | Handler |
| ----------- | ------ | ------- |
| `/api/projects/{slug}/content` | GET | `ListContent` (top-level) |
| `/api/projects/{slug}/content` | POST | `CreateContent` (top-level) |
| `/api/projects/{slug}/content/{parent}/children` | GET | `ListContent` (children of parent) |
| `/api/projects/{slug}/content/{parent}/children` | POST | `CreateContent` (under parent) |
| `/api/projects/{slug}/content/{parent}/reorder` | PUT | `ReorderContent` |
| `/api/projects/{slug}/content/{slug}/tree` | GET | `GetContentTree` |
| `/api/projects/{slug}/content/{slug-path...}` | GET | `GetContent` |
| `/api/projects/{slug}/content/{slug-path...}` | PUT | `UpdateContent` |
| `/api/projects/{slug}/content/{slug-path...}` | DELETE | `DeleteContent` |

## Request/Response DTOs

### `CreateContentRequest`

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `title` | `string` | yes | Content title |
| `slug` | `string` | no | Custom slug (auto-generated if empty) |
| `contentTypeId` | `string` | yes | ContentType UUID |
| `description` | `string` | no | Content description |
| `summary` | `string` | no | Content summary |
| `tags` | `[]string` | no | Tags array |
| `customFields` | `map[string]any` | no | Custom key-value pairs |
| `content` | `map[string]any` | no | ProseMirror JSON document |
| `isAutoTitled` | `bool` | no | Use auto-generated title |

### `UpdateContentRequest`

All fields are pointer types (`*string`, `*bool`, etc.) to support partial updates. Only non-nil fields are applied.

### `ContentResponse`

Full response DTO with all content fields, slug path, timestamps (formatted as ISO 8601), nested ContentType, children array, and child count.

## Delete with Glossary Cleanup

The `DeleteContent` handler performs glossary mark cleanup before deletion:

1. Loads the content item and its ProseMirror JSON
2. Calls `prosemirror.ExtractMarksOfType(content, "glossaryLink")` to find all glossary marks
3. Groups marks by `termId` and counts occurrences
4. Decrements `usageCount` and `docCount` on each affected glossary entry
5. Proceeds with deletion (best-effort: glossary cleanup failures are logged but don't block deletion)

## Helper Functions

| Function | Description |
| -------- | ----------- |
| `extractContentPath(path, projectSlug)` | Extract content subpath from full URL |
| `parseSlugPath(path)` | Split slash-separated path into string slice |
| `contentToResponse(dto)` | Convert repository DTO to API response |
| `contentTreeToResponse(dto)` | Convert tree DTO to API response (recursive) |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `repository` | internal | DTOs and input types |
| `prosemirror` | internal | `ExtractMarksOfType` for glossary cleanup |
| `sqlite` | internal | Type assertion for `DecrementGlossaryEntryCounts` |
| `request` | internal | Query params, body parsing, content-type validation |
| `response` | internal | OK, Created, NoContent, BadRequest, NotFound, InternalError |
| `contextx` | internal | `GetProjectSlugFromRequest` |
| `project` | internal | `Status` type |

## Side Effects

- Glossary count decrement on delete (best-effort, logged on failure)
