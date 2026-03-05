---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-frontend/search-results/","title":"SearchResults.svelte","tags":["file","search-frontend","component","panel"]}
---


# `SearchResults.svelte`

**Module:** `[[modules/search-frontend/overview|Search Frontend]]`
**Path:** `src/lib/components/search/SearchResults.svelte`

> [!NOTE] Panel component that displays search results. Registered in PanelRegistry, rendered by PanelContainer. Receives no props -- reads all state from the searchStore. Groups results by type (content, glossary) with count badges. Supports keyboard navigation and click-to-navigate.

## Panel Registration

Registered in `PanelRegistry.ts` with the following configuration:

| Property | Value |
| -------- | ----- |
| `id` | `'search-results'` |
| `name` | `'Search'` |
| `icon` | `'Search'` |
| `tier` | `'core'` |
| `alwaysEnabled` | `false` |
| `defaultPosition` | `'right'` |
| `allowedPositions` | `['left', 'right', 'bottom', 'floating']` |
| `defaultEnabled` | `true` |
| `order` | `10` |
| `source` | `'builtin'` |

The panel is auto-opened by the `ensureSearchPanelVisible()` helper in the search store when a search is initiated from the SearchBar.

## Props

This component receives **no props**. All state is read reactively from `$searchStore`.

## Exports

Exported via the barrel `src/lib/components/search/index.ts`:

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SearchResults` | component (default) | Panel search results display |

## Derived State

| Name | Type | Source | Description |
| ---- | ---- | ------ | ----------- |
| `searchState` | `SearchState` | `$searchStore` | Reactive snapshot of the full search state |
| `contentResults` | `SearchResultItem[]` | Filtered from `searchState.results` | Results where `type === 'content'` |
| `glossaryResults` | `SearchResultItem[]` | Filtered from `searchState.results` | Results where `type === 'glossary'` |
| `flatResults` | `SearchResultItem[]` | `[...contentResults, ...glossaryResults]` | Ordered list for keyboard navigation (content first, then glossary) |
| `hasMore` | `boolean` | `searchState.currentOffset < searchState.total` | Whether more pages of results are available |

## Local State

| Name | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| `focusedIndex` | `number` | `-1` | Index into `flatResults` of the keyboard-focused item. Reset to `0` when results change. |
| `containerRef` | `HTMLDivElement \| null` | `null` | Reference to the container element for scroll management |

## Visual States

The panel renders one of six mutually exclusive states:

| State | Condition | Display |
| ----- | --------- | ------- |
| **Idle** | No query, no results, not loading | Search icon + "Use the search bar..." hint + Ctrl+K shortcut |
| **Loading** | `isLoading` and no results yet | Spinner + "Searching..." |
| **Error** | `error` is non-null | AlertCircle icon + "Search Error" + error detail |
| **Message** | `message` is non-null and no results | Info icon + message text + "Try a different search query" hint |
| **Empty** | Not loading, no results, query present | Search icon + "No results found" + query echo |
| **Results** | Results array is non-empty | Summary bar + grouped results + optional "Load more" button |

When in the **Results** state, an optional message banner may also appear above the results (e.g., partial scope match info from the backend).

## Grouped Results Layout

Results are split into two groups, each with a sticky header:

1. **Content** group -- `FileText` icon, "CONTENT" label, count badge from `facets['content']`
2. **Glossary** group -- `BookOpen` icon, "GLOSSARY" label, count badge from `facets['glossary']`

Each result row is a `[[modules/search-frontend/SearchResultItem|SearchResultItem]]` component. The `isFocused` prop tracks keyboard focus, using the index within `flatResults` (content results use index `i`, glossary results use index `contentResults.length + i`).

## Keyboard Navigation

| Key | Action |
| --- | ------ |
| `ArrowDown` | Move focus to next result |
| `ArrowUp` | Move focus to previous result |
| `Home` | Jump to first result |
| `End` | Jump to last result |
| `Enter` | Activate the focused result (same as click) |

The container has `role="listbox"` and `tabindex="-1"` for focus management. The focused item is scrolled into view via `requestAnimationFrame` + `scrollIntoView({ block: 'nearest', behavior: 'smooth' })`.

Focus index resets to `0` (first result) whenever the `searchState.results` array reference changes (new search).

## Methods

| Method | Parameters | Returns | Description |
| ------ | ---------- | ------- | ----------- |
| `handleKeydown(event)` | `event: KeyboardEvent` | `void` | Keyboard navigation handler. Processes ArrowDown, ArrowUp, Enter, Home, End. |
| `scrollFocusedIntoView()` | none | `void` | Scrolls the `.result-item.focused` element into view using `requestAnimationFrame`. |
| `handleResultClick(result)` | `result: SearchResultItemType` | `Promise<void>` | Handles clicking a search result. Content results fetch the full Content object via `contentApi.getContent()` then dispatch `content:selected` via `dispatchContentSelected()`. Glossary results call `glossaryStore.navigateToEntry()`. |
| `handleLoadMore()` | none | `void` | Calls `searchStore.loadMore()` to fetch the next page of results. |

## Click Behavior by Result Type

### Content Results

1. Extract `slugPath` from the result and split into array (e.g., `"book-1/chapter-1/scene-1"` becomes `["book-1", "chapter-1", "scene-1"]`)
2. Fetch the full Content object from the backend via `contentApi.getContent(projectSlug, slugPathArray)` -- data source: SQLite database
3. Dispatch `content:selected` event via `dispatchContentSelected(content)` on the moduleEventBus
4. The editor page subscribes to `onContentSelected()` and loads the content into the editor

### Glossary Results

1. Call `glossaryStore.navigateToEntry(projectSlug, result.id)`
2. This ensures the glossary panel is visible, sets the project, and loads the entry detail view

## Pagination

The "Load more" button appears when `hasMore` is true (i.e., `currentOffset < total`). Clicking it calls `searchStore.loadMore()` which appends the next page to the existing results. The button shows the remaining count: `total - currentOffset`. While loading, the button displays a spinner and is disabled.

## Lifecycle

### onDestroy

When the SearchResults panel is unmounted (panel hidden or closed), `searchStore.clear()` is called. This:

- Resets all search state to defaults
- Clears the query, which causes the `$effect` in TextEditor.svelte to call `editor.setSearchHighlights('')`, removing all editor decorations

Panel components are unmounted from the DOM when hidden (not CSS `display:none`), so `onDestroy` fires reliably.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `onDestroy` | `svelte` | Lifecycle hook to clear store on unmount |
| `page` | `$app/stores` | Read `params.slug` for project context |
| `Search`, `FileText`, `BookOpen`, `AlertCircle`, `Info` | `lucide-svelte` | Icons for states and group headers |
| `searchStore` | `[[modules/search-frontend/search-store]]` | Read search state, call `loadMore` |
| `contentApi` | `$lib/api/content` | Fetch full Content object by slug path |
| `dispatchContentSelected` | `$lib/services/moduleEventBus` | Dispatch `content:selected` event |
| `glossaryStore` | `$lib/stores/glossary` | Navigate to glossary entry on click |
| `notificationStore` | `$lib/stores/notifications` | Show error notifications |
| `SearchResultItem` | `[[modules/search-frontend/SearchResultItem]]` | Child component for result rows |
| `SearchResultItemType` | `$lib/types/search` | TypeScript type for search result |
| `createLogger` | `$lib/services/loggerService` | Debug and info logging |

## Side Effects

- Calls `searchStore.clear()` on component destroy, which resets all search state and clears editor decorations.
- Fetches content objects from the backend API when content results are clicked.
- Dispatches `content:selected` events on the moduleEventBus.
- Calls `glossaryStore.navigateToEntry()` for glossary result clicks, which may open/show the glossary panel.

## Notes

> [!INFO] The panel reads the project slug from `$page.params.slug` for click handlers, not from a prop. This is because panel components in PanelContainer receive no props -- they must source context from stores or route params.

> [!WARNING] The `{@html}` rendering in SearchResultItem child components relies on the trust boundary that snippets are generated by the backend FTS5 engine, not user input.
