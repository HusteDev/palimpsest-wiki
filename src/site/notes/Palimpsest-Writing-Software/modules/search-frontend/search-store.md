---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-frontend/search-store/","title":"search.ts (Store & API)","tags":["file","search-frontend","store"],"updated":"2026-03-05T05:32:44.895-07:00"}
---


# `search.ts` (Store) & `search.ts` (API)

**Module:** `[[modules/search-frontend/overview|Search Frontend]]`
**Store Path:** `src/lib/stores/search.ts`
**API Path:** `src/lib/api/search.ts`

> [!NOTE] Reactive search state store and REST API client. The store acts as the bridge between SearchBar (in Header) and SearchResults (in PanelContainer) -- SearchBar writes search state, SearchResults reads it. Data source: Backend search API backed by per-project FTS5 index.

---

## Search Store (`src/lib/stores/search.ts`)

### Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SearchState` | interface | TypeScript interface for the full search state shape |
| `searchStore` | object | Svelte-subscribable store with `subscribe`, `search`, `loadMore`, `clear`, `setProject` methods |

### SearchState Interface

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `query` | `string` | `''` | Current search query string |
| `results` | `SearchResultItem[]` | `[]` | Search results from the API |
| `total` | `number` | `0` | Total result count across all pages (for pagination) |
| `facets` | `Record<string, number>` | `{}` | Per-type result counts, e.g. `{ content: 5, glossary: 3 }` |
| `isLoading` | `boolean` | `false` | True while an API call is in flight |
| `error` | `string \| null` | `null` | Error message if the last search failed |
| `message` | `string \| null` | `null` | Optional backend message (e.g. "Scope Not Found") |
| `currentOffset` | `number` | `0` | Number of results already loaded (pagination offset) |
| `limit` | `number` | `20` | Results per page (DEFAULT_LIMIT) |
| `projectSlug` | `string` | `''` | Current project slug (used to detect project changes) |

### Constants

| Name | Value | Description |
| ---- | ----- | ----------- |
| `SEARCH_PANEL_ID` | `'search-results'` | Panel ID used for auto-open logic |
| `DEFAULT_LIMIT` | `20` | Results per page for pagination |

### Store Methods

#### `search(projectSlug, query)`

Execute a search, replacing all current results and auto-opening the panel.

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `projectSlug` | `string` | Project to search within |
| `query` | `string` | Search query string (min 2 chars enforced by caller) |

**Returns:** `Promise<void>`

**Behavior:**
1. Sets `isLoading: true`, resets `error`, `message`, and `currentOffset` to 0
2. Calls `ensureSearchPanelVisible()` to auto-open/activate the search panel
3. Calls `searchApi.searchProject()` with the query and default limit/offset
4. On success: updates `results`, `total`, `facets`, `message`, `currentOffset`
5. On error: sets `error` message, clears `results`, `total`, `facets`

#### `loadMore()`

Load more results (pagination), appending to the existing results array.

**Returns:** `Promise<void>`

**Behavior:**
- No-op if already loading or if `currentOffset >= total`
- Calls `searchApi.searchProject()` with the current query and the `currentOffset` as pagination offset
- Appends new results to existing `results` array
- Updates `total`, `facets`, `currentOffset`

#### `clear()`

Clear all search state back to defaults.

**Returns:** `void`

**Behavior:**
- Resets the entire store to `defaultState`
- This triggers the `$effect` in TextEditor.svelte to clear editor search decorations (query becomes empty string)

#### `setProject(projectSlug)`

Handle project change. Clears search state if the project slug differs from the current search context.

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `projectSlug` | `string` | The new project slug |

**Returns:** `void`

### Helper: `ensureSearchPanelVisible()`

Internal (non-exported) function that ensures the search results panel is visible and active.

**Algorithm:**
1. Read `userPreferences.panelLayout` to find which panel position (left, right, bottom) contains `'search-results'` in its `components` array
2. If not found in any panel, add `'search-results'` to the right panel's components and set `visible: true`
3. If found but the panel is hidden, set `visible: true`
4. Switch the active tab of the target panel position to `'search-results'` via `userPreferences.updateUIState()`

### Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `writable`, `get` | `svelte/store` | Core store primitives |
| `searchApi` | `[[modules/search-frontend/search-store\|src/lib/api/search.ts]]` | REST API client |
| `userPreferences` | `$lib/stores/preferences` | Panel layout management for auto-open |
| `SearchResultItem` (type) | `$lib/types/search` | TypeScript type for results |
| `createLogger` | `$lib/services/loggerService` | Debug and info logging |

### Side Effects

- The `search()` method calls `ensureSearchPanelVisible()`, which mutates `userPreferences` to open/activate the search panel. This is the mechanism that auto-opens the results panel when the user types a search query.

---

## Search API Client (`src/lib/api/search.ts`)

### Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `searchProject` | function | Search across a project's content and glossary entries |
| `searchApi` | object | Grouped export containing `searchProject` (matches the `contentApi` pattern) |
| `SearchResponse` | type (re-export) | Full search response type |
| `SearchResultItem` | type (re-export) | Single result type |
| `SearchParams` | type (re-export) | Search parameter type |

### `searchProject(projectSlug, params)`

Search across a project's content and glossary entries. Calls `GET /api/projects/{slug}/search` with query parameters.

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `projectSlug` | `string` | The project slug to search within |
| `params` | `SearchParams` | Search parameters |

**Returns:** `Promise<SearchResponse>`

**Throws:** `Error` if the API call fails or returns `success: false`

#### SearchParams

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `query` | `string` | yes | Search query string (may include scope prefix like `"book-1:dragon"`) |
| `types` | `SearchResultType[]` | no | Optional type filter: `['content']`, `['glossary']`, or both |
| `limit` | `number` | no | Max results per page (default 20, max 100) |
| `offset` | `number` | no | Pagination offset (default 0) |

#### SearchResponse

| Field | Type | Description |
| ----- | ---- | ----------- |
| `results` | `SearchResultItem[]` | Ordered by BM25 relevance score |
| `total` | `number` | Total match count across all pages |
| `facets` | `Record<string, number>` | Per-type counts, e.g. `{ content: 30, glossary: 12 }` |
| `message` | `string?` | Optional info message (e.g. "Scope Not Found") |

#### SearchResultItem

| Field | Type | Description |
| ----- | ---- | ----------- |
| `type` | `'content' \| 'glossary'` | Result type discriminator |
| `id` | `string` | Entity UUID |
| `title` | `string` | Content title or glossary term |
| `snippet` | `string` | Highlighted excerpt with `<mark>` tags from FTS5 `snippet()` |
| `score` | `number` | BM25 relevance score (lower = more relevant) |
| `slugPath` | `string?` | Content slug path (e.g. `"series-1/book-1/chapter-1/scene-1"`). Absent for glossary results. |

### API Endpoint

```
GET /api/projects/{slug}/search?q={query}&types={types}&limit={limit}&offset={offset}
```

Query parameters are built using `URLSearchParams`. The `types` parameter is a comma-separated string (e.g., `"content,glossary"`).

### Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `apiClient` | `$lib/api/client` | Base HTTP client with error handling |
| `APIResponse` (type) | `$lib/types/project` | Generic API response wrapper |
| `SearchResponse`, `SearchParams` (types) | `$lib/types/search` | TypeScript types |

### Side Effects

None. Pure API call wrapper with no global state mutations.

## Notes

> [!INFO] The store uses a pattern of combining `subscribe` from a Svelte writable with named methods on a plain object. This allows `$searchStore` reactive syntax while providing a clean method API (`searchStore.search()`, `searchStore.clear()`, etc.).

> [!WARNING] The `ensureSearchPanelVisible()` helper directly mutates `userPreferences`. If the preferences store structure changes, this function must be updated accordingly.
