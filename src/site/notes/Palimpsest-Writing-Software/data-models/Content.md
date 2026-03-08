---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/data-models/content/","title":"Content","tags":["data-model","content"],"updated":"2026-03-05T06:54:53.998-07:00"}
---


# `Content`

> [!NOTE] Unified content entity that forms a recursive tree structure. Each item references a Project, a ContentType (which determines its level and capabilities), and optionally a parent Content. Only items whose ContentType has `hasContent=true` store ProseMirror JSON content.

## Fields

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `id` | UUID string | yes | auto-generated | Primary key |
| `title` | string | yes | — | Display title (auto-generated for auto-titled items) |
| `slug` | string | yes | — | URL-friendly identifier, derived from title |
| `description` | string | no | `""` | Optional description text |
| `summary` | string | no | `""` | Optional summary text |
| `order` | int | yes | `0` | Position within parent (0-indexed) |
| `word_count` | int | yes | `0` | Word count — direct for leaf, aggregated for containers |
| `is_locked` | bool | yes | `false` | Whether content is locked from editing |
| `is_auto_titled` | bool | yes | `false` | True if using auto-generated title (e.g., Scene-1) |
| `status` | Status enum | yes | `"draft"` | Content status (draft, editing, review, published, archived) |
| `tags` | []string (JSON) | no | `[]` | Tagging array |
| `custom_fields` | map[string]any (JSON) | no | `{}` | User-defined custom key-value pairs |
| `content` | map[string]any (JSON) | no | `{}` | ProseMirror JSON document — only for hasContent=true types |
| `created_at` | time.Time | yes | `time.Now` | Creation timestamp |
| `updated_at` | time.Time | yes | `time.Now` | Last update timestamp (auto-updated) |

## Edges (Relations)

| Edge | Target | Cardinality | Required | Description |
| ---- | ------ | ----------- | -------- | ----------- |
| `project` | [[data-models/Project\|Project]] | many-to-one | yes | Parent project |
| `content_type` | [[Palimpsest-Writing-Software/data-models/ContentType\|ContentType]] | many-to-one | yes | Content type defining level and capabilities |
| `parent` | Content | many-to-one | no | Parent content item (nil for top-level) |
| `children` | Content | one-to-many | — | Child content items |

## Constraints

- **Unique**: `(slug, parent)` — Slugs must be unique within their parent scope
- **Indexes**:
  - `(slug, parent)` — UNIQUE for slug resolution
  - `(order, parent)` — For efficient ordering queries
  - `(project, content_type)` — For filtering by type within project

## Read By

- [[Palimpsest-Writing-Software/modules/content-backend/contentRepo\|content.go (Repository)]] — `GetContent`, `GetContentByID`, `ListContent`, `ListAllContent`
- [[Palimpsest-Writing-Software/modules/content-backend/treeRepo\|tree.go]] — `GetContentTree`, `RecalculateWordCount`
- [[Palimpsest-Writing-Software/modules/content-backend/contentHandler\|content.go (Handler)]] — `DeleteContent` (reads for glossary mark extraction)

## Written By

- [[Palimpsest-Writing-Software/modules/content-backend/contentRepo\|content.go (Repository)]] — `CreateContent`, `UpdateContent`, `DeleteContent`, `UpdateContentBody`, `ReorderAndRenameSiblings`
- [[Palimpsest-Writing-Software/modules/content-backend/treeRepo\|tree.go]] — `ReorderContent`, `RecalculateWordCount`

## Transformations

- **Slug generation**: Title is slugified via `utils.Slugify()` on create if no custom slug is provided
- **Word count**: Recalculated from ProseMirror JSON on every content update; parent word counts are aggregated recursively
- **Auto-titling**: Items with `is_auto_titled=true` have their title and slug regenerated on reorder (two-pass approach to avoid UNIQUE violations)
- **FTS5 indexing**: Plain text extracted from ProseMirror JSON and indexed alongside title, tags, and slug path for full-text search

## Frontend DTO

The TypeScript `Content` interface (defined in [[Palimpsest-Writing-Software/modules/content-frontend/contentTypes\|content.ts]]) adds these frontend-only fields:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `body` | `string?` | HTML content (only if contentType.hasContent) |
| `slugPath` | `string[]?` | Full path from root |
| `contentType` | `ContentType?` | Populated content type info |
| `children` | `Content[]?` | Populated child items |
| `childCount` | `number?` | Number of direct children |
