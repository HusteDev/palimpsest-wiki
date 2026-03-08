---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-backend/content-types-seed/","title":"content_types_seed.go","tags":["file","content-backend","repository","seed"],"updated":"2026-03-05T06:25:23.435-07:00"}
---


# `content_types_seed.go`

**Module:** [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend]]
**Path:** `backend/internal/repository/sqlite/content_types_seed.go`

> [!NOTE] Seeds the default "book-series" content type hierarchy when a new project is created. Defines four levels: Series -> Book -> Chapter -> Scene, where only Scene has actual ProseMirror writing content.

## Exports (Store Methods)

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `seedContentTypes` | method | Create default hierarchy for a new project |

## Default Hierarchy: "book-series"

| Level | Name | hasContent | isSystem | Icon | Labels |
| ----- | ---- | ---------- | -------- | ---- | ------ |
| 1 | Series | false | true | `library` | singular: "Series", plural: "Series" |
| 2 | Book | false | true | `book` | singular: "Book", plural: "Books" |
| 3 | Chapter | false | true | `file-text` | singular: "Chapter", plural: "Chapters" |
| 4 | Scene | true | true | `square-pen` | singular: "Scene", plural: "Scenes" |

## Seed Process

1. Gets the project entity by ID
2. Creates each ContentType with parent linkage (Book parent -> Series, Chapter parent -> Book, Scene parent -> Chapter)
3. Links all four content types to the project via `proj.Update().AddContentTypes()`

> [!TIP] Only Level 4 (Scene) has `hasContent=true`. This means only Scenes store ProseMirror JSON and are directly editable in the text editor. All other levels are structural containers with aggregated word counts.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `ent` | generated | Ent client for ContentType creation |

## Side Effects

- Creates 4 ContentType records in the project database
- Links content types to the project entity
