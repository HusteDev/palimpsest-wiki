---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/get-occurrences/","title":"GET /glossary/{id}/occurrences","tags":["api","glossary"],"updated":"2026-03-05T05:32:37.819-07:00"}
---


# GET /glossary/{id}/occurrences

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler.getOccurrences()]]

> [!NOTE]
> Finds occurrences of a glossary term in content documents via FTS5 search.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |
| `id` | string | Entry UUID |

## Query Parameters

None.

## Request Body

None.

## Success Response

**Status:** `200 OK`

```json
{
  "term": "Arcane Rift",
  "occurrences": [
    {
      "contentId": "uuid-string",
      "title": "Chapter 3 - The Awakening",
      "snippet": "...the Arcane Rift opened above the city...",
      "slugPath": "chapter-3-the-awakening",
      "occurrenceCount": 4
    }
  ],
  "total": 12,
  "docCount": 3,
  "scanPending": false
}
```

The `scanPending` field is `true` when FTS5 finds documents but stored mark counts are 0 and `autoMark` is enabled, indicating that marks are currently being applied by a background job.

## Error Responses

| Status | Description |
| ------ | ----------- |
| 404 | Entry not found |
| 500 | Server error |

> [!NOTE]
> If the search provider is nil (graceful degradation), returns empty occurrences with zero counts.
