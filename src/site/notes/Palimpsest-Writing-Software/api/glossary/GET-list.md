---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/get-list/","title":"GET /glossary","tags":["api","glossary"]}
---


# GET /glossary

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler.listEntries()]]

> [!NOTE]
> Lists glossary entries with search, filtering, sorting, and pagination.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |

## Query Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `limit` | int | no | 50 | Results per page (1-100) |
| `offset` | int | no | 0 | Pagination offset |
| `search` | string | no | — | Case-insensitive on term or shortDefinition |
| `tags` | string | no | — | Comma-separated (any-match, post-query) |
| `category` | string | no | — | Exact match |
| `status` | string | no | — | Exact match |
| `sortBy` | string | no | `"term"` | `term` / `updatedAt` / `createdAt` |
| `sortOrder` | string | no | `"asc"` | `asc` / `desc` |

## Request Body

None.

## Success Response

**Status:** `200 OK`

```json
{
  "entries": [ /* GlossaryEntry[] */ ],
  "total": 42,
  "limit": 50,
  "offset": 0
}
```

**Schema:** [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|data-models/GlossaryEntryDTO]]

## Error Responses

| Status | Description |
| ------ | ----------- |
| 404 | Project not found |
| 500 | Server error |
