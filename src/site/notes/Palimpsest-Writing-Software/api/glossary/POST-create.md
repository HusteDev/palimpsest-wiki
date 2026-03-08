---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/post-create/","title":"POST /glossary","tags":["api","glossary"],"updated":"2026-03-05T05:32:38.037-07:00"}
---


# POST /glossary

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler.createEntry()]]

> [!NOTE]
> Creates a new glossary entry. Triggers a background term scan to apply marks in content.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |

## Request Body

JSON object. Required fields: `term` (non-empty string), `shortDefinition` (non-empty string). All other fields are optional. Boolean defaults: `autoMark = true`, `enableTooltip = true`, `enableHyperlink = false`.

```json
{
  "term": "Arcane Rift",
  "shortDefinition": "A tear in the fabric of reality caused by uncontrolled magic.",
  "category": "Magic System",
  "tags": ["magic", "worldbuilding"],
  "aliases": ["Rift", "Arcane Tear"]
}
```

## Success Response

**Status:** `201 Created`

Returns the full [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|GlossaryEntry]] object.

## Error Responses

| Status | Description |
| ------ | ----------- |
| 400 | Validation error or duplicate term (error message contains `"UNIQUE"`) |
| 404 | Project not found |
| 500 | Server error |

> [!TIP]
> After successful creation, a background `term_scan` job is submitted to apply glossaryLink marks in all in-scope content documents.
