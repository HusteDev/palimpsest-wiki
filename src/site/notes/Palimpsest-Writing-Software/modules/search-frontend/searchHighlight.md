---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-frontend/search-highlight/","title":"searchHighlight.ts","tags":["file","search-frontend","prosemirror","plugin"]}
---


# `searchHighlight.ts`

**Module:** `[[modules/search-frontend/overview|Search Frontend]]`
**Path:** `src/lib/components/editor/proseMirror/plugins/searchHighlight.ts`

> [!NOTE] ProseMirror plugin for temporary search match highlighting. Creates visual-only inline decorations (CSS class `.search-highlight`) that are never serialized to the document model. Designed with stub actions for future Find/Replace navigation.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `createSearchHighlightPlugin` | function | Factory that creates the ProseMirror plugin instance |
| `searchHighlightKey` | `PluginKey<SearchHighlightState>` | Unique key for retrieving plugin state and dispatching meta |
| `SearchHighlightMeta` | type | Discriminated union for transaction meta values |

## Data Flow

The search highlight system follows this reactive chain:

```
1. searchStore.query changes (user types in SearchBar)
2. TextEditor.svelte $effect fires (reactive subscription to $searchStore.query)
3. TextEditor calls editor.setSearchHighlights(query)
4. ProseMirrorEditor.setSearchHighlights() dispatches:
     tr.setMeta(searchHighlightKey, { action: 'setQuery', query })
5. Plugin apply() scans document, creates Decoration.inline() for each match
6. ProseMirror renders decorations as visual overlays (CSS class .search-highlight)
```

When search is cleared (query becomes empty string), step 5 produces `DecorationSet.empty` and all highlights disappear.

## Types

### MatchPosition

Internal interface for a single match location in the document.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `from` | `number` | Document offset where the match starts |
| `to` | `number` | Document offset where the match ends |

### SearchHighlightMeta (exported)

Discriminated union for transaction meta values. External code dispatches these via `tr.setMeta(searchHighlightKey, meta)`.

| Action | Fields | Description |
| ------ | ------ | ----------- |
| `setQuery` | `{ action: 'setQuery'; query: string }` | Set a new search query. Scans the entire document and highlights all matches. |
| `next` | `{ action: 'next' }` | **STUB** -- Move to next match. Will be implemented with Find/Replace feature. |
| `previous` | `{ action: 'previous' }` | **STUB** -- Move to previous match. Will be implemented with Find/Replace feature. |
| `goToMatch` | `{ action: 'goToMatch'; index: number }` | **STUB** -- Jump to a specific match by index. Will be implemented with Find/Replace feature. |

### SearchHighlightState

Internal plugin state managed by the search highlight plugin.

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `query` | `string` | `''` | Current search query string |
| `matches` | `MatchPosition[]` | `[]` | All match positions in document order (stored for future Find/Replace iteration) |
| `activeMatchIndex` | `number` | `-1` | Index of the currently focused match. **STUB** -- not used yet, will be activated with Find/Replace. |
| `decorations` | `DecorationSet` | `DecorationSet.empty` | Decoration set containing inline decorations for all matches |

## Plugin Key

```typescript
export const searchHighlightKey = new PluginKey<SearchHighlightState>('searchHighlight');
```

Used for two purposes:
- **Retrieve state:** `searchHighlightKey.getState(editorState)` returns the current `SearchHighlightState`
- **Dispatch meta:** `tr.setMeta(searchHighlightKey, meta)` sends commands to the plugin

## `buildDecorations(doc, query)`

Scans a ProseMirror document for all occurrences of a query string and builds inline decorations for each match.

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `doc` | `ProseMirrorNode` | The ProseMirror document node to scan |
| `query` | `string` | The search string to find (case-insensitive) |

**Returns:** `{ matches: MatchPosition[]; decorations: DecorationSet }`

**Algorithm:**
1. If query is empty or whitespace-only, return empty matches and `DecorationSet.empty`
2. Convert query to lowercase for case-insensitive matching
3. Walk all nodes via `doc.descendants()`, processing only text nodes
4. For each text node, find all occurrences of the lowercase query in the lowercase text
5. For each match, record the `{ from, to }` position and create `Decoration.inline(from, to, { class: 'search-highlight' })`
6. Return the matches array and a `DecorationSet.create(doc, decorations)`

## `createSearchHighlightPlugin()`

Factory function that creates the ProseMirror plugin instance.

**Returns:** `Plugin<SearchHighlightState>`

### Plugin State Machine (`state.init` / `state.apply`)

#### `init()`

Returns the default state with empty query, no matches, `activeMatchIndex: -1`, and `DecorationSet.empty`.

#### `apply(tr, oldState, _oldEditorState, newEditorState)`

Processes each transaction through three decision branches:

| Condition | Action |
| --------- | ------ |
| **Meta present** (`setQuery`) | Scan the new document with the new query via `buildDecorations()`. Reset `activeMatchIndex` to -1. |
| **Meta present** (`next` / `previous` / `goToMatch`) | **STUB** -- log and return unchanged state. |
| **Document changed** and query is non-empty | Re-scan the new document with the existing query (handles content swap when user clicks a search result). |
| **No meta, no doc change**, decorations exist | Map existing decorations through the transaction mapping (`decorations.map(tr.mapping, tr.doc)`). This efficiently handles cursor movement and selection changes without a full re-scan. |
| **No meta, no doc change**, decorations empty | Return unchanged state. |

### Plugin Props

#### `decorations(state)`

Returns the `DecorationSet` from the plugin state for ProseMirror to render.

```typescript
decorations(state: EditorState): DecorationSet {
    const pluginState = searchHighlightKey.getState(state);
    return pluginState?.decorations ?? DecorationSet.empty;
}
```

## Integration Points

### ProseMirrorEditor

The plugin is instantiated in `ProseMirrorEditor.ts` constructor as part of the plugins array:

```typescript
createSearchHighlightPlugin(),  // Visual-only decorations for search match highlighting
```

`ProseMirrorEditor` exposes three public methods for interacting with the plugin:

| Method | Description |
| ------ | ----------- |
| `setSearchHighlights(query)` | Dispatches `{ action: 'setQuery', query }` meta. Pass empty string to clear all highlights. |
| `nextSearchMatch()` | **STUB** -- Will dispatch `{ action: 'next' }` meta. |
| `previousSearchMatch()` | **STUB** -- Will dispatch `{ action: 'previous' }` meta. |

### TextEditor.svelte

A reactive `$effect` in TextEditor.svelte bridges the search store to the editor:

```typescript
$effect(() => {
    const query = $searchStore.query;
    if (editor) {
        editor.setSearchHighlights(query);
    }
});
```

This fires whenever `$searchStore.query` changes, including when `searchStore.clear()` resets the query to an empty string (which clears all decorations).

### CSS Styles

Defined in TextEditor.svelte's `<style>` block:

| CSS Class | Selector | Styles |
| --------- | -------- | ------ |
| `.search-highlight` | `:global(.ProseMirror .search-highlight)` | `background: var(--editor-find-highlight, #fef3c7); border-radius: 2px;` |
| `.search-highlight-active` | `:global(.ProseMirror .search-highlight-active)` | `background: var(--editor-find-highlight-active, #f59e0b); border-radius: 2px;` **STUB** -- not yet applied by the plugin. |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Plugin`, `PluginKey` | `prosemirror-state` | ProseMirror plugin infrastructure |
| `Decoration`, `DecorationSet` | `prosemirror-view` | Visual-only inline decorations |
| `EditorState`, `Transaction` (types) | `prosemirror-state` | TypeScript types for apply() signature |
| `Node` (type, aliased as `ProseMirrorNode`) | `prosemirror-model` | Document node type for buildDecorations() |
| `createLogger` | `$lib/services/loggerService` | Debug logging |

## Side Effects

None at module level. The plugin registers itself into ProseMirror's plugin system when instantiated, but that is controlled by the caller (ProseMirrorEditor constructor).

## Notes

> [!INFO] Decorations are visual-only (`Decoration.inline`) and are never serialized to the document JSON. They exist purely in ProseMirror's view layer and are discarded when the editor is destroyed or when the query is cleared.

> [!WARNING] The `next`, `previous`, and `goToMatch` actions are stubs. They log a debug message and return unchanged state. These will be implemented when the Find/Replace feature is built. The `activeMatchIndex` field and `.search-highlight-active` CSS class are pre-wired for that future feature.

> [!INFO] The `buildDecorations` function performs case-insensitive matching by lowercasing both the query and each text node's content. This matches the backend FTS5 behavior which is also case-insensitive.
