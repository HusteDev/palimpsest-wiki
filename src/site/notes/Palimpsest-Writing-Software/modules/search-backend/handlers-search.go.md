---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-backend/handlers-search-go/","title":"handlers/search.go","tags":["file","search-backend"]}
---


# `handlers/search.go`

**Module:** [[Palimpsest-Writing-Software/modules/search-backend/overview\|Search Backend]]
**Path:** `backend/internal/http/handlers/search.go`

> [!NOTE] HTTP handler for the unified search endpoint. Parses query parameters, delegates to the `SearchProvider`, and returns ranked results with snippet highlighting, facet counts, and graceful degradation on provider errors.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SearchHandler` | struct | HTTP handler with search provider and logger fields |
| `NewSearchHandler` | function | Constructor: `(provider SearchProvider)` returns `*SearchHandler` |

### SearchHandler Struct

```go
type SearchHandler struct {
    provider search.SearchProvider
    log      *logging.Logger
}
```

## Key Methods

### Search

```
Search(w http.ResponseWriter, r *http.Request)
```

Handles `GET /api/projects/{slug}/search` requests. This is the single entry point for all search API requests.

**Query parameters:**

| Parameter | Required | Default | Description |
| --------- | -------- | ------- | ----------- |
| `q` | yes | -- | Search query. May include scope prefix (e.g. `"book-1:dragon"`). |
| `types` | no | all | Comma-separated type filter: `"content"`, `"glossary"`. Invalid values silently dropped. |
| `limit` | no | 20 | Maximum results to return. Capped at 100. |
| `offset` | no | 0 | Pagination offset. |

**Response:** JSON body matching `search.SearchResult`:

```go
type SearchResult struct {
    Results []SearchResultItem `json:"results"`
    Total   int                `json:"total"`
    Facets  map[string]int     `json:"facets"`
    Message string             `json:"message,omitempty"`
}

type SearchResultItem struct {
    Type     string  `json:"type"`
    ID       string  `json:"id"`
    Title    string  `json:"title"`
    Snippet  string  `json:"snippet"`
    Score    float64 `json:"score"`
    SlugPath string  `json:"slugPath,omitempty"`
}
```

**Error handling and graceful degradation:**

| Condition | HTTP Status | Behavior |
| --------- | ----------- | -------- |
| Missing `q` parameter | 400 Bad Request | Returns error message |
| Missing project slug | 400 Bad Request | Returns error message |
| Non-GET method | 405 Method Not Allowed | Returns error |
| No results found | 200 OK | Empty `results` array, `total: 0` |
| Scope not found | 200 OK | Empty results with `message: "Scope Not Found"` |
| Provider error (any) | 200 OK | Empty results with `message: "Search temporarily unavailable"` |

The graceful degradation pattern ensures the frontend always receives a valid JSON response, even when the search provider fails. Provider errors are logged at ERROR level but the HTTP response is always 200 with empty results. This follows 12-Factor principles: search is treated as a backing service that may be unavailable.

**Execution flow:**

1. Validate HTTP method is GET
2. Extract `projectSlug` from request context via `contextx.GetProjectSlugFromRequest`
3. Validate required `q` parameter
4. Parse `types` parameter: split by comma, validate each against `"content"` and `"glossary"` whitelist
5. Parse `limit` parameter: default 20, enforce maximum of 100
6. Parse `offset` parameter: default 0, must be non-negative
7. Log debug entry with all parsed parameters
8. Build `search.SearchOptions` and call `provider.Search`
9. On provider error: log error, return empty results with degradation message
10. Return `200 OK` with JSON result via `response.Success`

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `search` | `backend/internal/search` | `SearchProvider` interface, `SearchOptions`, `SearchResult`, `SearchResultItem` types |
| `contextx` | `backend/internal/http/contextx` | Extract `projectSlug` from request context |
| `response` | `backend/internal/http/response` | HTTP response helpers (`Success`, `BadRequest`, `MethodNotAllowed`) |
| `logging` | `backend/internal/logging` | Structured logging with `"handler-search"` component |
| `net/http` | stdlib | HTTP types and method constants |
| `strconv` | stdlib | Integer parsing for `limit` and `offset` parameters |
| `strings` | stdlib | `TrimSpace`, `Split` for parameter parsing |

## Side Effects

- Reads from FTS5 virtual tables via the `SearchProvider` (read-only database operations)
- Logs at DEBUG level for every search request and at ERROR level for provider failures

## Notes

> [!WARNING] The `types` parameter validation silently drops invalid values. If a client sends `types=content,invalid,glossary`, only `content` and `glossary` are used. If all values are invalid, the filter becomes empty (equivalent to searching all types). This is intentional to avoid breaking clients that may send future type values.
