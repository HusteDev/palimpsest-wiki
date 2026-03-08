---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/content-types/","title":"content.ts (Types)","tags":["file","content-frontend","types"],"updated":"2026-03-05T06:23:17.786-07:00"}
---


# `content.ts` (Types)

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/types/content.ts`

> [!NOTE] TypeScript type definitions for the hierarchical content system. Defines Content, ContentType, ProseMirror JSON shapes, and all API request/response interfaces.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ProseMirrorMarkJSON` | interface | Serialized PM mark (`type`, optional `attrs`) |
| `ProseMirrorNodeJSON` | interface | Serialized PM node (`type`, optional `attrs`, `content`, `marks`, `text`) |
| `ProseMirrorDocJSON` | type | Alias for ProseMirrorNodeJSON — root document |
| `ContentType` | interface | Content type definition (id, name, hierarchyName, level, hasContent, isSystem, metadata) |
| `ContentTypeMetadata` | interface | Display metadata (icon, color, pluralLabel, singularLabel, slugStrategy) |
| `Content` | interface | Content item (id, title, slug, description, summary, order, wordCount, isLocked, isAutoTitled, status, tags, customFields, body, content, slugPath, contentType, children, childCount) |
| `ContentHierarchy` | interface | Hierarchy definition (name, types array, contentLevel) |
| `CreateContentInput` | interface | Create request (title, slug?, contentTypeId, description?, summary?, tags?, customFields?, content?, isAutoTitled?) |
| `UpdateContentInput` | interface | Update request (all fields optional: title, slug, description, summary, body, wordCount, tags, customFields, content, status, isLocked, isAutoTitled, order) |
| `ListContentResponse` | interface | Paginated list response (content array, total, limit, offset) |
| `ReorderContentInput` | interface | Reorder request with items array of { id, order } |
| `ListContentOptions` | interface | List filter options (limit, offset, search, status, contentTypeId, includeChildren) |
| `PluginContentJSON` | type | Generic JSON for plugin content (`Record<string, unknown>`) |

## Key Interfaces

### `Content`

The central type representing a content item in the hierarchy. Key fields:
- `body?: string` — HTML content for editing (only if contentType.hasContent)
- `content?: ProseMirrorDocJSON` — ProseMirror JSON (only if contentType.hasContent)
- `slugPath?: string[]` — Full path from root (e.g., `['book-1', 'part-1', 'chapter-1']`)
- `contentType?: ContentType` — Populated content type info
- `children?: Content[]` — Populated child content items
- `isAutoTitled: boolean` — True if using auto-generated title (e.g., Scene-1)

### `UpdateContentInput`

Uses optional fields (`content?: ProseMirrorDocJSON | PluginContentJSON`) to support both ProseMirror editor content and generic plugin JSON.

## Imports / Dependencies

None — pure type definitions with no runtime imports.

## Side Effects

None.
