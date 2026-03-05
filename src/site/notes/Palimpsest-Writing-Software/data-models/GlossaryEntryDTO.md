---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/data-models/glossary-entry-dto/","title":"GlossaryEntryDTO","tags":["data-model","glossary","dto"]}
---


# GlossaryEntryDTO

> [!NOTE]
> Data transfer objects and TypeScript types for glossary entry data flowing between backend repository, HTTP handler, and frontend client.

- **Source files:** `backend/internal/repository/dto.go`, `src/lib/types/glossary.ts`

---

## GlossaryEntryDTO (Go)

Maps 1:1 with [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|GlossaryEntry]] schema fields (excluding `index_text`). All `[]string` slices are guaranteed non-nil (empty array instead of null in JSON). Used as the return type from all repository CRUD methods.

---

## CreateGlossaryEntryInput (Go)

Input struct for creating a new glossary entry.

- **Required:** `Term`, `ShortDefinition`
- **All other fields:** optional
- **Slice fields:** default to `nil`
- **Boolean/string fields:** default to Go zero values
- The HTTP handler applies editor marking defaults: `AutoMark=true`, `EnableTooltip=true`, `EnableHyperlink=false`

---

## UpdateGlossaryEntryInput (Go)

Input struct for partial updates to an existing glossary entry.

All fields are pointers (`*string`, `*bool`, `*[]string`, `*map[string]any`). Nil pointers = skip during update. Only non-nil fields are applied to the Ent update builder.

---

## ListGlossaryEntriesInput (Go)

Pagination and filtering parameters for listing glossary entries.

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `Limit` | int | 50 | Page size (1-100) |
| `Offset` | int | 0 | Pagination offset |
| `Search` | string | — | Case-insensitive match on `term` or `short_definition` |
| `Tags` | []string | — | Any-match post-query filter |
| `Category` | string | — | Exact match on category |
| `Status` | string | — | Exact match on status |
| `SortBy` | string | `"term"` | Sort column |
| `SortOrder` | string | `"asc"` | `asc` or `desc` |

---

## ListGlossaryEntriesResult (Go)

| Field | Type | Description |
| --- | --- | --- |
| `Entries` | `[]GlossaryEntryDTO` | Page of results |
| `Total` | int | Total matching entries (before pagination) |

---

## TermOccurrenceDTO (Go)

Represents a single occurrence of a glossary term found in project content.

| Field | Type | Description |
| --- | --- | --- |
| `ContentID` | string | ID of the content item containing the occurrence |
| `ContentTitle` | string | Title of the content item |
| `Context` | string | Surrounding text snippet |
| `Position` | int | Character offset within the content |

---

## GlossaryEntry (TypeScript)

Defined in `src/lib/types/glossary.ts`. Frontend interface mirroring the Go [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|GlossaryEntryDTO]] with camelCase field names and TypeScript union types.

### Union Types

| Type Name | Values |
| --- | --- |
| `GlossaryTermStatus` | `'draft'` / `'review'` / `'approved'` / `'deprecated'` |
| `TermScope` | `'project'` / `'series'` / `'book'` / `'chapter'` |
| `SpoilerLevel` | `'none'` / `'mild'` / `'major'` |
| `GlossaryCategory` | `'Characters'` / `'Locations'` / `'Items'` / `'Lore'` / `'Concepts'` / `'Events'` / `'Organizations'` |
| `PartOfSpeech` | `'noun'` / `'verb'` / `'adjective'` / `'concept'` / `'process'` / `'artifact'` / `'role'` / `'system'` |

---

## Read By

- [[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler]] (all handler methods)
- [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-api\|glossaryApi]] (all API client methods)
- [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]] (state management)

## Written By

- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|glossaryEntryToDTO()]] (Ent entity -> DTO)
- [[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|glossaryEntryToResponse()]] (DTO -> API JSON)
