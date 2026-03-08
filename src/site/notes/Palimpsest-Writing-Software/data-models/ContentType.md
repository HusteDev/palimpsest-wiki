---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/data-models/content-type/","title":"ContentType","tags":["data-model","content-type"],"updated":"2026-03-05T06:27:14.429-07:00"}
---


# `ContentType`

> [!NOTE] Defines the structural levels of a content hierarchy. Each ContentType has a level number, a hierarchy name, and a `hasContent` flag indicating whether items of this type store actual ProseMirror writing content. Forms a self-referential parent-child chain (Series > Book > Chapter > Scene).

## Fields

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `id` | UUID string | yes | auto-generated | Primary key |
| `name` | string | yes | — | Display name (e.g., "Chapter", "Scene") |
| `hierarchy_name` | string | yes | — | Hierarchy identifier (e.g., "book-series") |
| `level` | int (positive) | yes | — | Level in hierarchy (1 = top, increments down) |
| `has_content` | bool | yes | `false` | True if this level stores ProseMirror JSON |
| `is_system` | bool | yes | `false` | True for built-in types, false for user-created |
| `metadata` | map[string]any (JSON) | no | `{}` | UI metadata (icon, color, labels, slugStrategy) |
| `created_at` | time.Time | yes | `time.Now` | Creation timestamp |
| `updated_at` | time.Time | yes | `time.Now` | Last update timestamp |

## Metadata Schema

The `metadata` JSON field stores UI display information:

| Key | Type | Description |
| --- | ---- | ----------- |
| `icon` | string | Lucide icon name (e.g., "library", "book", "file-text", "square-pen") |
| `color` | string | Hex color for display (optional) |
| `singularLabel` | string | Singular display name (e.g., "Chapter") |
| `pluralLabel` | string | Plural display name (e.g., "Chapters") |
| `slugStrategy` | string | Slug generation strategy: "custom", "order", "uuid", "auto" |

## Edges (Relations)

| Edge | Target | Cardinality | Required | Description |
| ---- | ------ | ----------- | -------- | ----------- |
| `parent` | ContentType | many-to-one | no | Parent type in hierarchy |
| `children` | ContentType | one-to-many | — | Child types in hierarchy |
| `content_items` | [[Palimpsest-Writing-Software/data-models/Content\|Content]] | one-to-many | — | Content items using this type |

## Constraints

- **Unique**: `(hierarchy_name, level)` — Only one type per level per hierarchy
- **Indexes**:
  - `(hierarchy_name, level)` — UNIQUE for hierarchy structure
  - `(hierarchy_name)` — For querying all types in a hierarchy

## Default Hierarchy: "book-series"

Seeded by [[Palimpsest-Writing-Software/modules/content-backend/contentTypesSeed\|content_types_seed.go]] on project creation:

| Level | Name | hasContent | Parent |
| ----- | ---- | ---------- | ------ |
| 1 | Series | false | — |
| 2 | Book | false | Series |
| 3 | Chapter | false | Book |
| 4 | Scene | true | Chapter |

## Read By

- [[Palimpsest-Writing-Software/modules/content-backend/contentRepo\|content.go (Repository)]] — `GetContentTypesByHierarchy`, `GetContentType`
- [[Palimpsest-Writing-Software/modules/content-backend/contentHandler\|content.go (Handler)]] — via `contentTypeToResponse`

## Written By

- [[Palimpsest-Writing-Software/modules/content-backend/contentRepo\|content.go (Repository)]] — `CreateContentType`
- [[Palimpsest-Writing-Software/modules/content-backend/contentTypesSeed\|content_types_seed.go]] — `seedContentTypes` (on project creation)

## Frontend DTO

The TypeScript `ContentType` interface (in [[Palimpsest-Writing-Software/modules/content-frontend/contentTypes\|content.ts]]):

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | UUID |
| `name` | `string` | Display name |
| `hierarchyName` | `string` | Hierarchy identifier (camelCase) |
| `level` | `number` | Level in hierarchy |
| `hasContent` | `boolean` | Whether this level has editable content |
| `isSystem` | `boolean` | Built-in vs user-created |
| `parentId` | `string?` | Parent type UUID |
| `metadata` | `ContentTypeMetadata` | Typed metadata object |
