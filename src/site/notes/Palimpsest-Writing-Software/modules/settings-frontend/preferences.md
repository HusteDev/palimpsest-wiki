---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/settings-frontend/preferences/","title":"preferences.ts","tags":["file","settings-frontend","typescript","svelte","store"],"updated":"2026-03-05T08:59:54.405-07:00"}
---


# `preferences.ts`

**Module:** [[Palimpsest-Writing-Software/modules/settings-frontend/overview\|Settings Frontend]]
**Path:** `src/lib/stores/preferences.ts`

> [!NOTE] Mutable Svelte store for user preferences including theme, font, locale, editor settings, word count display, panel layout (docked and floating), and hidden UI state. Provides load/save via the backend API, debounced saves for rapid layout changes, legacy format migration, and a complete floating panel lifecycle (pop-out, dock, move, resize, minimize, close, z-index management).

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `UserPreferences` | interface | Complete user preferences shape |
| `EditorSettings` | interface | Editor-specific settings (spellcheck, autosave, etc.) |
| `WordCountSettings` | interface | Word count display and formatting options |
| `UIState` | interface | Hidden UI state persisted across sessions |
| `PanelConfig` | interface | Configuration for a single docked panel position |
| `FloatingPanelInstance` | interface | Position, size, and state of a floating panel |
| `PanelLayout` | interface | Full panel layout combining docked and floating panels |
| `userPreferences` | object | Primary export: subscribable store with all action methods |

## `UserPreferences` Interface

Complete user preferences. All fields are persisted to the backend via `PUT /api/settings`.

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `theme` | `string` | `'light'` | Active theme identifier |
| `font` | `'default' \| 'dyslexic'` | `'default'` | Font family preference |
| `locale` | `string` | `'en'` | UI language locale code |
| `panelLayout` | `PanelLayout` | see below | Full docked + floating panel layout |
| `disableAutoNumbering` | `boolean` | `false` | When `true`, hides Auto-Title option and prevents automatic slug numbering |
| `editor` | `EditorSettings` | see below | Editor-specific settings |
| `wordCount` | `WordCountSettings` | see below | Word count display settings |
| `lastProjectSlug` | `string` | `''` | Slug of the last opened project for session restoration |
| `uiState` | `UIState` | see below | Hidden UI state persisted across sessions |

## `EditorSettings` Interface

Editor-specific settings controlling the text editing experience.

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `spellcheck` | `boolean` | `true` | Enable browser spellcheck |
| `grammarCheck` | `boolean` | `true` | Enable browser grammar checking |
| `autosave` | `boolean` | `true` | Enable periodic auto-save to backend |
| `autosaveInterval` | `number` | `5` | Minutes between auto-saves |
| `wordWrap` | `boolean` | `true` | Enable word wrap in the editor |
| `saveOnNavigation` | `'prompt' \| 'enable' \| 'disable'` | `'prompt'` | Behavior when navigating away from unsaved content: `"prompt"` shows a modal, `"enable"` saves silently, `"disable"` keeps draft in localStorage |

## `WordCountSettings` Interface

Controls word count display in the content tree and status bar.

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `showInTree` | `boolean` | `true` | Show word counts in the content navigation tree |
| `hierarchyVisibility` | `Record<string, boolean>` | `{}` | Per-hierarchy level visibility keyed by `contentTypeId`. If not specified for a level, defaults to `showInTree` value |
| `showWordCount` | `boolean` | `true` | Show word count in the status bar |
| `showCharCount` | `boolean` | `true` | Show character count in the status bar |
| `showPageEstimate` | `boolean` | `false` | Show page estimate in the status bar |
| `charCountMode` | `'with-spaces' \| 'without-spaces'` | `'with-spaces'` | Whether character count includes spaces |
| `numberFormat` | `'locale' \| 'plain' \| 'compact'` | `'locale'` | Formatting style for displayed numbers |
| `wordsPerPage` | `number` | `250` | Words per page for page estimate calculation (industry standard) |

## `UIState` Interface

Hidden UI state that is persisted across sessions but not directly user-configurable through a settings form. Supports dynamic keys for extensibility (e.g., `activeTab_left`, `activeTab_right`).

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `expandedTreeIds` | `string[]` | `[]` | Content IDs of expanded tree nodes |
| `collapsedPanels` | `{ left: boolean; right: boolean; bottom: boolean }` | all `false` | Collapsed state per docked panel position (`true` = collapsed to mini mode) |
| `[key: string]` | `unknown` | -- | Dynamic keys for panel system state (e.g., `activeTab_left`) |

## `PanelConfig` Interface

Configuration for a single docked panel position (left, right, or bottom).

| Field | Type | Default (left) | Default (right) | Default (bottom) | Description |
| ----- | ---- | --------------- | ---------------- | ----------------- | ----------- |
| `width` | `number` | `300` | `300` | `0` | Panel width in pixels |
| `height` | `number` | `0` | `0` | `200` | Panel height in pixels |
| `visible` | `boolean` | `true` | `true` | `false` | Whether the panel is currently shown |
| `enabled` | `boolean` | `true` | `true` | `true` | Whether the panel position is enabled |
| `components` | `string[]` | `['content-navigation']` | `['glossary', 'search-results']` | `[]` | IDs of registered panel components assigned to this position |

## `FloatingPanelInstance` Interface

Configuration for a single floating panel instance. Each floating panel tracks its position, dimensions, and z-order independently.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `panelId` | `string` | Unique instance ID (UUID v4 format, generated by `generatePanelId()`) |
| `componentId` | `string` | Reference to registered component ID from PanelRegistry |
| `x` | `number` | Viewport X position in pixels |
| `y` | `number` | Viewport Y position in pixels |
| `width` | `number` | Panel width in pixels |
| `height` | `number` | Panel height in pixels |
| `zIndex` | `number` | Stacking order (higher = on top). Base value is `500` |
| `minimized` | `boolean` | Whether panel is collapsed to title bar only |

## `PanelLayout` Interface

Complete layout configuration combining docked and floating panels.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `left` | `PanelConfig` | Left docked panel configuration |
| `right` | `PanelConfig` | Right docked panel configuration |
| `bottom` | `PanelConfig` | Bottom docked panel configuration |
| `floating` | `FloatingPanelInstance[]` | Array of all floating panel instances |

## Constants

### `defaultPreferences`

Full default `UserPreferences` object used as the initial store value, the fallback on load failure, and the reset target. See the default values listed in each interface table above.

### `FLOATING_PANEL_BASE_Z`

```ts
const FLOATING_PANEL_BASE_Z = 500;
```

Base z-index for floating panels, matching the CSS variable `--z-floating-panel`. Used by `normalizeZIndices()` as the starting point when reassigning z-indices.

## Internal Stores

| Store | Type | Initial Value | Description |
| ----- | ---- | ------------- | ----------- |
| `preferences` | `writable<UserPreferences>` | `defaultPreferences` | Primary internal store holding current preferences |
| `isLoading` | `writable<boolean>` | `false` | Whether a load operation is in progress |
| `error` | `writable<string \| null>` | `null` | Last error message from load or save, or `null` |

## Internal State

| Variable | Type | Initial Value | Description |
| -------- | ---- | ------------- | ----------- |
| `saveTimeout` | `ReturnType<typeof setTimeout> \| null` | `null` | Handle for the debounce timer used by `debouncedSave()` |
| `nextZIndex` | `number` | `FLOATING_PANEL_BASE_Z` (500) | Monotonically increasing counter for assigning z-indices to floating panels |

## `migratePanelLayout` Behavior

```
migratePanelLayout(raw: any) -> PanelLayout
```

Converts legacy panel layout formats to the current `PanelLayout` shape. Called during `loadPreferences()` to handle preferences saved before the Panel System was introduced.

**Migration logic per position (left, right, bottom):**

1. If the position is missing in the raw data, uses defaults for that position.
2. Resolves `components` via a fallback chain: non-empty `components` array > non-empty `addons` array (legacy field) > default components for the position. Empty arrays do not short-circuit the chain, because the old format stored `addons: []` for positions that had hardcoded components.
3. Maps `width`, `height`, `visible`, and `enabled` with defaults for any missing fields.
4. For `floating`: uses the raw array if it is an array, otherwise defaults to `[]` (handles preferences saved before floating panels existed).

## `loadPreferences` Behavior

```
loadPreferences() -> Promise<void>
```

Fetches user preferences from the backend API and applies migrations for legacy formats.

**Data source:** Backend API (`GET /api/settings` -> JSON file)

**Steps:**

1. Sets `isLoading` to `true` and clears `error`.
2. Fetches `GET /api/settings` via `apiClient`.
3. **Panel layout migration** -- Passes raw `panelLayout` through `migratePanelLayout()`.
4. **UIState migration** -- Ensures `collapsedPanels` exists (handles preferences saved before the collapse/mini-mode feature).
5. **WordCount migration** -- Applies defaults for any missing `wordCount` fields individually using `??` fallback.
6. Maps all remaining top-level fields (`theme`, `font`, `locale`, `disableAutoNumbering`, `editor`, `lastProjectSlug`) with `||` fallback to defaults.
7. Sets the `preferences` store with the fully migrated object.
8. On failure: logs a warning, sets `error`, and keeps defaults.
9. Sets `isLoading` to `false` in the `finally` block.

## `savePreferences` Behavior

```
savePreferences(prefs: UserPreferences) -> Promise<void>
```

Persists the full preferences object to the backend.

**Data destination:** Backend API (`PUT /api/settings` -> JSON file)

Serializes all top-level fields (`theme`, `font`, `locale`, `panelLayout`, `disableAutoNumbering`, `editor`, `wordCount`, `lastProjectSlug`, `uiState`) and sends them via `apiClient.put()`. On failure, logs the error and sets the `error` store.

## `debouncedSave` Behavior

```
debouncedSave() -> void
```

Wraps `savePreferences()` in a debounce timer using the `PANEL_DEBOUNCE` constant from `$lib/config`. If called again before the timer fires, the previous timer is cleared and a new one is started. Used by panel layout updates, UI state updates, and panel collapse actions to prevent excessive API calls during drag-resize operations.

## Theme, Font, and Locale Updaters

| Function | Signature | Save | Description |
| -------- | --------- | ---- | ----------- |
| `updateTheme` | `(theme: string) -> void` | immediate | Updates and saves the theme preference |
| `updateFont` | `(font: 'default' \| 'dyslexic') -> void` | immediate | Updates and saves the font preference |

Both functions update the internal store and then call `savePreferences()` immediately (no debounce).

## `updatePanelLayout` Behavior

```
updatePanelLayout(updates: { left?: Partial<PanelConfig>; right?: Partial<PanelConfig>; bottom?: Partial<PanelConfig> }) -> void
```

Applies partial updates to docked panel configurations. For each position provided in `updates`, merges the update with the existing config. Width and height values are rounded to integers via `Math.round()` when present. Uses `debouncedSave()` because this is typically called during drag-resize operations that fire rapidly.

## `togglePanel` Behavior

```
togglePanel(panel: 'left' | 'right' | 'bottom') -> void
```

Toggles the `visible` flag on the specified docked panel position and saves immediately (not debounced). This is a discrete user action (clicking a menu item) rather than a continuous operation.

## Panel Collapse Actions (Mini Mode)

These actions manage the collapsed/expanded state of docked panels. When collapsed, a panel shows only an icon strip (mini mode). All collapse actions use `debouncedSave()`.

| Function | Signature | Description |
| -------- | --------- | ----------- |
| `collapsePanel` | `(position) -> void` | Sets `collapsedPanels[position]` to `true` in `uiState` |
| `expandPanel` | `(position, activeTabId?) -> void` | Sets `collapsedPanels[position]` to `false`. Optionally sets `activeTab_{position}` in `uiState` to focus a specific tab |
| `togglePanelCollapsed` | `(position) -> void` | Reads current state and delegates to `collapsePanel` or `expandPanel` |
| `isPanelCollapsed` | `(position) -> boolean` | Synchronous read of collapsed state. Returns `false` if `collapsedPanels` is missing |

## Editor and Word Count Updaters

| Function | Signature | Save | Description |
| -------- | --------- | ---- | ----------- |
| `updateEditor` | `(settings: Partial<EditorSettings>) -> void` | immediate | Merges partial editor settings and saves |
| `updateWordCount` | `(settings: Partial<WordCountSettings>) -> void` | immediate | Merges partial word count settings and saves |

Both accept partial objects and merge them with the existing settings using the spread operator.

## Other Updaters

| Function | Signature | Save | Description |
| -------- | --------- | ---- | ----------- |
| `setLastProject` | `(slug: string) -> void` | immediate | Records the last opened project slug for session restoration |
| `setDisableAutoNumbering` | `(disabled: boolean) -> void` | immediate | Toggles auto-numbering. When disabled, hides Auto-Title option and prevents automatic slug numbering |
| `updateUIState` | `(updates: Partial<UIState>) -> void` | debounced | Merges partial UI state updates. Uses `debouncedSave()` because UI state changes can be frequent (tree expansion, tab switching) |
| `resetPreferences` | `() -> void` | immediate | Resets the entire preferences store to `defaultPreferences` and saves |

## Floating Panel Actions

These actions manage the lifecycle of floating (detached) panels. Floating panels are independent windows within the application viewport that can be moved, resized, minimized, and stacked.

### `generatePanelId`

```
generatePanelId() -> string
```

Generates a UUID v4 string using `Math.random()`. Used internally to assign unique IDs to new floating panel instances.

### `popOutPanel`

```
popOutPanel(componentId: string, sourcePosition: 'left' | 'right' | 'bottom', initialBounds?: { x?, y?, width?, height? }) -> FloatingPanelInstance | null
```

Pops a docked component out into a floating panel.

**Steps:**

1. Verifies the component exists in the source position's `components` array. Returns `null` with a `console.warn` if not found.
2. Calculates default position (centered in viewport) and size (400x300) or uses `initialBounds` if provided.
3. Creates a `FloatingPanelInstance` with a generated `panelId`, the next z-index, and `minimized: false`.
4. Removes the component from the source position's `components` array.
5. Appends the new instance to `panelLayout.floating`.
6. Saves immediately.
7. Returns the created `FloatingPanelInstance`.

### `dockPanel`

```
dockPanel(panelId: string, targetPosition: 'left' | 'right' | 'bottom') -> void
```

Docks a floating panel back to a docked position. Finds the floating panel by `panelId`, removes it from the `floating` array, and appends its `componentId` to the target position's `components` array. Logs a `console.warn` if the panel is not found. Saves immediately.

### `updateFloatingPanel`

```
updateFloatingPanel(panelId: string, updates: Partial<Pick<FloatingPanelInstance, 'x' | 'y' | 'width' | 'height' | 'minimized'>>) -> void
```

Updates a floating panel's position and/or size. All numeric values are rounded via `Math.round()`. Uses `debouncedSave()` because this is typically called during drag/resize operations.

### `closeFloatingPanel`

```
closeFloatingPanel(panelId: string) -> void
```

Removes a floating panel from the layout. The component is **not** automatically re-docked -- it is simply removed. Saves immediately.

### `bringToFront`

```
bringToFront(panelId: string) -> void
```

Raises a floating panel to the top of the stacking order by assigning the next available z-index from the `nextZIndex` counter. If `nextZIndex` exceeds 900 (approaching the modal z-index of 1000), triggers `normalizeZIndices()` to compress the range. Uses `debouncedSave()`.

### `normalizeZIndices`

```
normalizeZIndices() -> void
```

Prevents z-index overflow by reassigning all floating panel z-indices starting from `FLOATING_PANEL_BASE_Z` (500) while preserving relative stacking order. Sorts panels by current z-index, then reassigns sequential values. Resets `nextZIndex` to `FLOATING_PANEL_BASE_Z + count`.

### `toggleMinimize`

```
toggleMinimize(panelId: string) -> void
```

Toggles the `minimized` flag on a floating panel (collapsed to title bar only vs. full panel). Saves immediately.

### `getFloatingPanel`

```
getFloatingPanel(panelId: string) -> FloatingPanelInstance | undefined
```

Synchronous lookup of a floating panel by `panelId`. Returns the `FloatingPanelInstance` or `undefined` if not found.

## `userPreferences` Exported Object

The primary public API. Combines store subscription, loading state, error state, and all action methods.

| Member | Kind | Description |
| ------ | ---- | ----------- |
| `subscribe` | store method | Svelte store contract -- subscribe to `UserPreferences` changes |
| `isLoading` | sub-store | Subscribe to loading state |
| `error` | sub-store | Subscribe to last error message |
| **Core Actions** | | |
| `load` | method | Alias for `loadPreferences()` |
| `updateTheme` | method | Update and save theme |
| `updateFont` | method | Update and save font |
| `updatePanelLayout` | method | Update docked panel dimensions (debounced) |
| `togglePanel` | method | Toggle docked panel visibility (immediate) |
| `updateEditor` | method | Update editor settings (immediate) |
| `updateWordCount` | method | Update word count settings (immediate) |
| `setLastProject` | method | Record last opened project (immediate) |
| `setDisableAutoNumbering` | method | Toggle auto-numbering (immediate) |
| `updateUIState` | method | Update hidden UI state (debounced) |
| `reset` | method | Alias for `resetPreferences()` |
| **Floating Panel Actions** | | |
| `popOutPanel` | method | Pop docked component into floating panel |
| `dockPanel` | method | Dock floating panel back to position |
| `updateFloatingPanel` | method | Update position/size (debounced) |
| `closeFloatingPanel` | method | Remove floating panel (immediate) |
| `bringToFront` | method | Raise z-index to top (debounced) |
| `toggleMinimize` | method | Toggle minimize state (immediate) |
| `getFloatingPanel` | method | Get floating panel by ID (synchronous) |
| **Panel Collapse Actions** | | |
| `collapsePanel` | method | Collapse to mini mode (debounced) |
| `expandPanel` | method | Expand from mini mode (debounced) |
| `togglePanelCollapsed` | method | Toggle collapsed state |
| `isPanelCollapsed` | method | Check if collapsed (synchronous) |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `writable`, `derived`, `get` | `svelte/store` | Reactive store primitives and synchronous value extraction |
| `apiClient` | `$lib/api/client` | HTTP client for `GET /api/settings` and `PUT /api/settings` |
| `PANEL_DEBOUNCE` | `$lib/config` | Debounce interval constant for panel layout saves |
| `createLogger` | `$lib/services/loggerService` | Logger instance (component name: `'preferences'`) |

## Side Effects

- Creates a module-level logger via `createLogger('preferences')` used for warning and error messages during load/save
- Module-level mutable state (`saveTimeout`, `nextZIndex`) is shared across all callers within the same JS context
- `debouncedSave()` uses `setTimeout` / `clearTimeout` which means rapid successive calls consolidate into a single API write
- `popOutPanel()` reads `window.innerWidth` and `window.innerHeight` to calculate centered default positions for new floating panels
- `normalizeZIndices()` mutates z-index values of all floating panels when triggered by `bringToFront()`

## Notes

> [!WARNING] The `migratePanelLayout()` function handles two legacy format migrations: (1) old `addons` field renamed to `components`, and (2) missing `floating` array. These migrations are not "backwards compatibility" features -- they handle preferences saved during earlier development iterations and will remain until the initial release stabilizes the format.

> [!WARNING] The `loadPreferences()` and `loadSettings()` (in `systemSettings.ts`) both call `GET /api/settings` on the same endpoint. This means preferences and system settings are loaded from the same API response but split across two stores. Any change to the API response shape must be coordinated across both files.
