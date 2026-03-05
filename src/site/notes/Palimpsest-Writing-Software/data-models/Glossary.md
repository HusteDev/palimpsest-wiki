---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/data-models/glossary/","title":"Glossary","tags":["data-model","glossary"]}
---


# Glossary

> [!NOTE]
> Thin parent entity that links a collection of glossary entries to a project. Auto-created on first access.

- **Schema file:** `backend/internal/ent/schema/Glossary.go`
- **Database table:** `glossaries`
- **Edge to Project:** `edge.From("project", Project.Type).Ref("glossary").Unique().Required()` (1:1 with Project)
- **Edge to entries:** `edge.To("entries", GlossaryEntry.Type)` (1:N to [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|GlossaryEntry]])

---

## Fields

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `id` | string | yes | UUID auto | Primary key |
| `name` | string | yes | `"Glossary"` | Display name |
| `description` | string | no | null | Optional description |
| `created_at` | time | yes | `time.Now` | Auto-set on create |
| `updated_at` | time | yes | `time.Now` | Auto-set on create and update |
| `project_glossary` | FK string | yes | — | Unique FK to `projects.id` |

---

## Constraints

- **Unique:** `project_glossary` (one glossary per project)
- **Foreign key:** `project_glossary` -> `projects.id`

---

## Read By

- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|getOrCreateGlossary()]]

## Written By

- [[Palimpsest-Writing-Software/modules/glossary-backend/glossary.go\|getOrCreateGlossary()]] (lazy creation)

---

## Transformations

Created lazily by `getOrCreateGlossary()` — if a project has no glossary when a glossary operation is requested, one is created with name `"Glossary"` and linked to the project.
