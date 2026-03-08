---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/search/get-search/","title":"GET Search","tags":["api","search"],"updated":"2026-03-05T05:32:38.274-07:00"}
---


# GET /search

## Handler

[[Palimpsest-Writing-Software/modules/search-backend/handlers-search.go\|SearchHandler.Search()]]

> [!NOTE]
> Executes a unified full-text search across content documents and glossary entries, returning ranked results with snippet highlighting and facet counts.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |

## Query Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `q` | string | yes | -- | Search query. May include a scope prefix (e.g. `book-1:dragon`). |
| `types` | string | no | all types | Comma-separated type filter. Valid values: `content`, `glossary`. |
| `limit` | integer | no | 20 | Maximum results to return (1--100). Values above 100 are clamped. |
| `offset` | integer | no | 0 | Pagination offset. |

## Request Body

None.

## Request Example

```bash
curl "http://localhost:3001/api/projects/my-novel/search?q=dragon&types=content,glossary&limit=20&offset=0"
```

## Success Response

**Status:** `200 OK`

```json
{
  "results": [
    {
      "type": "content",
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "title": "Chapter 3 - The Dragon's Lair",
      "snippet": "The <mark>dragon</mark> stirred in the depths of the mountain...",
      "score": -8.42,
      "slugPath": "series-1/book-1/chapter-3"
    },
    {
      "type": "glossary",
      "id": "f9e8d7c6-b5a4-3210-fedc-ba0987654321",
      "title": "Dragon",
      "snippet": "A large winged <mark>dragon</mark> native to the northern peaks...",
      "score": -7.15
    }
  ],
  "total": 2,
  "facets": {
    "content": 1,
    "glossary": 1
  }
}
```

### Response Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `results` | array | Ordered list of `SearchResultItem` objects, sorted by relevance score. |
| `results[].type` | string | Result type: `"content"` or `"glossary"`. |
| `results[].id` | string | Entity UUID. |
| `results[].title` | string | Content document title or glossary term. |
| `results[].snippet` | string | Highlighted excerpt with `<mark>` tags around matched terms. |
| `results[].score` | number | FTS5 BM25 rank score. Lower (more negative) values indicate higher relevance. |
| `results[].slugPath` | string | Content slug path (e.g. `"series-1/book-1/chapter-3"`). Empty for glossary results. |
| `total` | integer | Total number of matches across all types (used for pagination). |
| `facets` | object | Per-type match counts, keyed by type name (e.g. `{"content": 30, "glossary": 12}`). |
| `message` | string | Optional informational message. Present only when the server needs to communicate additional context (e.g. `"Scope Not Found"`). |

**Schema:** [[Palimpsest-Writing-Software/data-models/SearchResult\|data-models/SearchResult]]

## Error Responses

| Status | Condition | Body |
| ------ | --------- | ---- |
| `400` | Missing or empty `q` parameter | `{ "error": "search query 'q' is required" }` |
| `400` | Missing project slug | `{ "error": "project slug is required" }` |
| `405` | Non-GET method used | `{ "error": "method not allowed" }` |

> [!NOTE] Graceful Degradation
> When the search provider encounters an internal error, the endpoint returns `200 OK` with an empty results array and the message `"Search temporarily unavailable"` rather than a 500 error. This allows the frontend to display a user-friendly message without triggering error-handling flows.

> [!TIP] Scope Not Found
> When the query includes a scope prefix (e.g. `book-1:dragon`) and the specified scope cannot be resolved, the response includes `"message": "Scope Not Found"` with a `200` status and empty results. The frontend should surface this message so the user knows the scope path was invalid rather than that no results matched.

## Query Syntax

| Syntax | Example | Description |
| ------ | ------- | ----------- |
| Simple terms | `dragon` | Matches documents containing the term. |
| Phrases | `"fire dragon"` | Matches the exact phrase. |
| Wildcards | `drag*` | Prefix matching via FTS5 prefix queries. |
| Scoped | `book-1:dragon` | Restricts search to a specific scope path. |
| NOT | `dragon NOT ice` | Excludes documents containing the negated term. |
