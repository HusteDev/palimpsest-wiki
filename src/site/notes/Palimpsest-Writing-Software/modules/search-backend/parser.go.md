---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-backend/parser-go/","title":"search/parser.go","tags":["file","search-backend"],"updated":"2026-03-05T05:32:44.414-07:00"}
---


# `search/parser.go`

**Module:** [[Palimpsest-Writing-Software/modules/search-backend/overview\|Search Backend]]
**Path:** `backend/internal/search/parser.go`

> [!NOTE] Parses raw search input into a structured `ParsedQuery` with scope extraction, quoted phrase handling, wildcard detection, and FTS5 compatibility routing.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ParseQuery` | function | Parses raw search input into `*ParsedQuery` |
| `ParsedQuery` | struct | Structured result of parsing: scope, term, wildcard flags, FTS5 flag |
| `ErrEmptyQuery` | variable | Sentinel error for empty/whitespace-only input |
| `ErrEmptyTerm` | variable | Sentinel error for scoped query with no term after the colon |

### ParsedQuery Struct

```go
type ParsedQuery struct {
    ScopePath        []string // Slug path segments, e.g. ["series-1", "book-1"]. Empty if no scope.
    ScopeRaw         string   // Original scope string before splitting, e.g. "series-1/book-1".
    SearchTerm       string   // The search term after scope extraction.
    HasScope         bool     // True if the query includes a scope prefix.
    HasWildcardScope bool     // True if any scope segment contains '*'.
    UseFTS5          bool     // True if FTS5 MATCH can handle this query natively.
}
```

## Key Functions

### ParseQuery

```
ParseQuery(raw string) (*ParsedQuery, error)
```

Parses a raw search input string into a structured `ParsedQuery`. This is the entry point for all search query processing.

**Parameters:**
- `raw` -- the user's search input, e.g. `"book-1:dragon*"` or `"fire dragon"`

**Returns:**
- `*ParsedQuery` -- parsed result with scope, term, and compatibility flags
- `error` -- `ErrEmptyQuery` if input is empty, `ErrEmptyTerm` if scoped query has no term

**Parsing rules (applied in order):**

1. Trim whitespace. Return `ErrEmptyQuery` if empty.
2. If the entire input is double-quoted (e.g. `"fire dragon"`), strip quotes and use as search term with no scope.
3. Find the first unquoted `:` by scanning character-by-character and tracking quote state.
4. Left of `:` is the scope. Split by `/`, filter empty segments. Check for `*` in any segment to set `HasWildcardScope`.
5. Right of `:` is the search term. Trim whitespace. Return `ErrEmptyTerm` if empty.
6. No `:` found means the entire input is the search term (no scope).
7. Determine `UseFTS5`: if `*` only appears at the end of term tokens, FTS5 MATCH can handle it natively (prefix wildcard). If `*` appears at the start or middle of any token, LIKE fallback is required.

**Query examples:**

| Input | ScopePath | SearchTerm | HasScope | UseFTS5 |
| ----- | --------- | ---------- | -------- | ------- |
| `dragon` | `[]` | `dragon` | false | true |
| `drag*` | `[]` | `drag*` | false | true |
| `*dragon` | `[]` | `*dragon` | false | false |
| `dr*gon` | `[]` | `dr*gon` | false | false |
| `"fire dragon"` | `[]` | `fire dragon` | false | true |
| `book-1:dragon` | `["book-1"]` | `dragon` | true | true |
| `series/book:dragon` | `["series", "book"]` | `dragon` | true | true |
| `book-*:dragon` | `["book-*"]` | `dragon` | true | true |
| `dragon NOT castle` | `[]` | `dragon NOT castle` | false | true |

### isValidFTS5Query

```
isValidFTS5Query(term string) bool
```

Checks whether a search term can be handled entirely by FTS5 MATCH syntax. Splits the term on whitespace and checks each token for wildcard position.

**FTS5 supports:**
- Plain terms: `dragon` -> `MATCH 'dragon'`
- Prefix wildcards: `drag*` -> `MATCH 'drag*'`
- Phrases: `fire dragon` -> `MATCH '"fire dragon"'`
- Boolean: `dragon NOT castle` -> `MATCH 'dragon NOT castle'`

**FTS5 does NOT support (requires LIKE fallback):**
- Suffix wildcards: `*dragon` -> requires `LIKE '%dragon'`
- Infix wildcards: `dr*gon` -> requires `LIKE 'dr%gon'`

**Parameters:**
- `term` -- the search term (already extracted from scope)

**Returns:** `true` if FTS5 can handle the query natively

## Internal Helpers

| Name | Description |
| ---- | ----------- |
| `findUnquotedColon` | Scans string for the first `:` not inside double quotes. Returns byte index or -1. |
| `splitScopeSegments` | Splits scope string by `/` and filters empty segments (handles leading/trailing/double slashes). |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `errors` | stdlib | `errors.New` for sentinel errors |
| `strings` | stdlib | `TrimSpace`, `Fields`, `Split`, `Contains`, `Trim`, `Index` for parsing |

## Side Effects

None. `ParseQuery` is a pure function with no side effects, global state access, or I/O.

## Notes

> [!WARNING] The colon-based scope separator means that literal colons in search terms (e.g. searching for `http://example.com`) will be interpreted as scope prefixes. If `http` does not match any content slug, the scope resolution will return `ErrScopeNotFound` and the search will return zero results with a `"Scope Not Found"` message. Users must double-quote such terms to avoid this: `"http://example.com"`.
