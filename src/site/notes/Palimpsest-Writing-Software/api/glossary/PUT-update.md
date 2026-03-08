---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/put-update/","title":"PUT /glossary/{id}","tags":["api","glossary"],"updated":"2026-03-05T05:32:38.171-07:00"}
---


# PUT /glossary/{id}

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler.updateEntry()]]

> [!NOTE]
> Updates an existing glossary entry. Only non-null fields are applied. Triggers background term scan.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |
| `id` | string | Entry UUID |

## Request Body

JSON object. All fields are optional (null or omitted means no change). Must include at least one non-null field. `term` and `shortDefinition` must be non-empty strings if provided.

```json
{
  "shortDefinition": "Updated definition with more detail.",
  "tags": ["magic", "worldbuilding", "danger"],
  "autoMark": false
}
```

## Success Response

**Status:** `200 OK`

Returns the updated full [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|GlossaryEntry]] object.

## Error Responses

| Status | Description |
| ------ | ----------- |
| 400 | No fields provided, or empty `term` / `shortDefinition` |
| 404 | Entry not found |
| 500 | Server error |

> [!TIP]
> Relationship changes trigger reciprocal updates on target entries. A background `term_scan` job re-scans content for updated term/aliases/scope.
