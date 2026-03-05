---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/search-frontend/search-bar/","title":"SearchBar.svelte","tags":["file","search-frontend","component"]}
---


# `SearchBar.svelte`

**Module:** `[[modules/search-frontend/overview|Search Frontend]]`
**Path:** `src/lib/components/search/SearchBar.svelte`

> [!NOTE] Search input component rendered in the Header toolbar. Provides debounced search input, Ctrl/Cmd+K keyboard shortcut to focus, Escape to clear, and a loading spinner. Writes search state to the searchStore; results are rendered separately by the SearchResults panel component.

## Props

| Prop | Type | Required | Default | Description |
| ---- | ---- | -------- | ------- | ----------- |
| `projectSlug` | `string` | yes | -- | Project slug to search within. Passed from the editor page route parameter. |

## Exports

This component is exported via the barrel `src/lib/components/search/index.ts`:

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SearchBar` | component (default) | Header search input |

## Local State

| Name | Type | Description |
| ---- | ---- | ----------- |
| `query` | `string` | Current input value, bound to the text field |
| `inputRef` | `HTMLInputElement \| null` | Reference to the input element for programmatic focus |
| `debounceTimer` | `ReturnType<typeof setTimeout> \| null` | Active debounce timer handle |
| `isLoading` | `boolean` (derived) | Derived from `$searchStore.isLoading`, drives the spinner |

## Constants

| Name | Value | Description |
| ---- | ----- | ----------- |
| `DEBOUNCE_MS` | `300` | Milliseconds to wait after last input before firing the search API call |
| `MIN_QUERY_LENGTH` | `2` | Minimum trimmed query length before a search is triggered |

## Methods

| Method | Parameters | Returns | Description |
| ------ | ---------- | ------- | ----------- |
| `handleInput()` | none | `void` | Input event handler. Clears the previous debounce timer, validates minimum query length, and sets a new debounce timer that calls `searchStore.search()`. If the trimmed query is empty, calls `searchStore.clear()`. |
| `handleKeydown(event)` | `event: KeyboardEvent` | `void` | Keydown handler on the input. **Enter** fires the search immediately (bypasses debounce). **Escape** clears the query, resets the store, and blurs the input. |
| `clearSearch()` | none | `void` | Resets query to empty string, clears the debounce timer, and calls `searchStore.clear()`. |
| `handleGlobalKeydown(event)` | `event: KeyboardEvent` | `void` | Global keydown listener registered on `<svelte:window>`. **Ctrl+K** (Windows/Linux) or **Cmd+K** (Mac) focuses and selects the search input. |

## Keyboard Shortcuts

| Shortcut | Scope | Action |
| -------- | ----- | ------ |
| `Ctrl+K` / `Cmd+K` | Global (window) | Focus and select the search input |
| `Enter` | Input focused | Fire search immediately, bypassing debounce |
| `Escape` | Input focused | Clear search query, blur input |

## Reactive Effects

| Effect | Trigger | Action |
| ------ | ------- | ------ |
| Project change | `projectSlug` prop changes | Calls `searchStore.setProject(projectSlug)` to clear stale results from a different project |
| Cleanup | Component destroy | Clears the debounce timer via return callback |

## Responsive Behavior

The entire `.search-container` is hidden below 768px viewport width via a CSS media query:

```css
@media (max-width: 768px) {
    .search-container {
        display: none;
    }
}
```

On mobile viewports, search is not available through the header bar. A future mobile search flow may be added.

## Visual States

| Condition | Display |
| --------- | ------- |
| Idle, empty input | Search icon + placeholder text + `Ctrl+K` keyboard hint badge |
| Typing, query present | Search icon + input text + X (clear) button |
| Loading | Animated Loader2 spinner replaces search icon |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Search`, `Loader2`, `X` | `lucide-svelte` | Icon components |
| `searchStore` | `[[modules/search-frontend/search-store]]` | Read loading state, dispatch search/clear/setProject |
| `createLogger` | `$lib/services/loggerService` | Debug and info logging |

## Side Effects

- Registers a global `keydown` listener on `<svelte:window>` for the Ctrl/Cmd+K shortcut. Removed automatically when the component unmounts.
- Calls `searchStore.setProject()` reactively when `projectSlug` changes, which may clear existing search state.

## Notes

> [!WARNING] The search bar is hidden on mobile (`max-width: 768px`). No alternative mobile search entry point exists yet.

> [!INFO] Enter bypasses debounce for instant feedback. This is intentional -- users pressing Enter expect immediate results, not a 300ms delay.
