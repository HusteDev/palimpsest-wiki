---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/data-models/glossary-entry/","title":"GlossaryEntry","tags":["data-model","glossary"],"updated":"2026-03-05T05:32:38.588-07:00"}
---


# GlossaryEntry

> [!NOTE]
> Per-project Ent ORM entity storing glossary terms with 37 fields covering definitions, categorization, linguistic metadata, scoping, relationships, editor marking, and usage tracking.

- **Schema file:** `backend/internal/ent/schema/GlossaryEntry.go`
- **Database table:** `glossary_entries`
- **Edge:** `edge.From("glossary", Glossary.Type).Ref("entries").Unique().Required()` — each entry belongs to exactly one [[Palimpsest-Writing-Software/data-models/Glossary\|Glossary]] (M:1, required)

---

## Fields

### Core Identification

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `id` | string | yes | UUID auto | Primary key |
| `term` | string | yes | — | The glossary term (NotEmpty validator) |
| `slug` | string | yes | — | URL-safe identifier (NotEmpty, unique per glossary) |
| `status` | string | yes | `"draft"` | `draft` / `review` / `approved` / `deprecated` |

### Definitions

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `short_definition` | string | yes | `""` | Plain-text tooltip/list definition |
| `full_definition` | JSON map[string]any | no | null | ProseMirror JSON document for rich text |

### Categorization

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `category` | string | no | null | `Characters` / `Locations` / `Items` / `Lore` / `Concepts` / `Events` / `Organizations` |
| `tags` | JSON []string | no | null | Search/filter labels |
| `aliases` | JSON []string | no | null | Alternative names, abbreviations |

### Linguistic Metadata

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `part_of_speech` | string | no | null | `noun` / `verb` / `adjective` / `concept` / `process` / `artifact` / `role` / `system` |
| `pronunciation_guide` | string | no | null | IPA notation |

### Scoping

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `term_scope` | string | yes | `"project"` | `project` / `series` / `book` / `chapter` |
| `scope_targets` | JSON []string | no | null | Slug paths of content items this term applies to |
| `world_scope` | string | no | null | Culture, kingdom, faction, era, region |
| `spoiler_level` | string | yes | `"none"` | `none` / `mild` / `major` |

### Examples and Usage

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `examples` | JSON []string | no | null | Correct usage examples |
| `non_examples` | JSON []string | no | null | Common misunderstandings |
| `usage_notes` | string | no | null | Style guidance, casing, pluralization |
| `etymology` | string | no | null | Historical or in-world origin |

### Visual Identity

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `icon` | string | no | null | Lucide icon name or emoji |
| `image_url` | string | no | null | Reference image URL |

### Editor Marking

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `auto_mark` | bool | yes | `true` | Auto-mark term occurrences in content |
| `mark_color` | string | yes | `""` | CSS color for underline |
| `mark_hover_color` | string | yes | `""` | CSS hover background color |
| `enable_tooltip` | bool | yes | `true` | Show short definition on hover |
| `enable_hyperlink` | bool | yes | `false` | Click mark navigates to entry |

### Usage Counts

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `usage_count` | int | yes | `0` | Total glossaryLink marks across all documents |
| `doc_count` | int | yes | `0` | Number of distinct documents with marks |

### Relationships

All relationship fields are JSON `[]string` (optional). Values are entry IDs referencing other [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|GlossaryEntry]] records.

| Field | Inverse | Description |
| --- | --- | --- |
| `broader_terms` | target's `narrower_terms` | Parent/hypernym concepts (asymmetric) |
| `narrower_terms` | target's `broader_terms` | Child/hyponym subconcepts (asymmetric) |
| `related_terms` | target's `related_terms` | Associative links (symmetric) |
| `synonyms` | target's `synonyms` | Equivalent meanings (symmetric) |
| `antonyms` | target's `antonyms` | Contrasts/opposites (symmetric) |
| `see_also` | target's `see_also` | Cross-references (symmetric) |

### Internal / Timestamps

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `index_text` | text | no | computed | Lowercase composite of term + aliases + shortDef + tags for FTS5 |
| `created_at` | time | yes | `time.Now` | Auto-set on create |
| `updated_at` | time | yes | `time.Now` | Auto-set on create and update |

---

## Constraints

- **Unique:** `(slug, glossary_entries FK)` composite index
- **Indexed:** `status`, `category`, `term` (non-unique)
- **Validators:** `term` (NotEmpty), `slug` (NotEmpty)
- **Foreign key:** `glossary_entries` -> [[Palimpsest-Writing-Software/data-models/Glossary\|Glossary]].id

---

## Read By

- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store.GetGlossaryEntry()]]
- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store.ListGlossaryEntries()]]
- [[Palimpsest-Writing-Software/modules/glossary-backend/term_scan_handler.go\|TermScanHandler.entryLoader()]]

## Written By

- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store.CreateGlossaryEntry()]]
- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store.UpdateGlossaryEntry()]]
- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store.DeleteGlossaryEntry()]]
- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|Store.UpdateGlossaryEntryCounts()]]

---

## Transformations

- **`generateSlug(term)`**: lowercase -> spaces to hyphens -> strip non-alnum -> collapse hyphens
- **`ensureUniqueSlug()`**: appends `-2`, `-3` etc. if slug exists within the glossary
- **`buildIndexText()`**: joins term + shortDef + aliases + tags lowercased for FTS5
- **`glossaryEntryToDTO()`**: ensures all `[]string` slices are non-nil (empty array vs null in JSON)
- **Reciprocal relationship sync**: adding/removing IDs in relationship arrays triggers inverse updates on target entries
