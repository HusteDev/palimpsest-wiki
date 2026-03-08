---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/data-models/panel-layout/","title":"PanelLayout","tags":["data-model","panels"],"updated":"2026-03-05T05:32:38.840-07:00"}
---


# `PanelLayout`

> [!NOTE] Composite data model defining the entire panel system's persisted state: docked panel configurations for left/right/bottom positions, and an array of active floating panel instances. Stored as a nested object within the user preferences JSON file.

## `PanelLayout`

The top-level container for all panel state.

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `left` | `PanelConfig` | yes | Left sidebar configuration |
| `right` | `PanelConfig` | yes | Right sidebar configuration |
| `bottom` | `PanelConfig` | yes | Bottom bar configuration |
| `floating` | `FloatingPanelInstance[]` | yes | Active floating panel instances (default: `[]`) |

## `PanelConfig`

Configuration for a single docked panel position.

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `width` | `number` | yes | `300` | Panel width in pixels (left/right) or height (bottom) |
| `height` | `number` | yes | `200` | Panel height in pixels (used by bottom panel) |
| `visible` | `boolean` | yes | `true` | Whether the panel region is shown |
| `enabled` | `boolean` | yes | `true` | Whether the panel region is active |
| `components` | `string[]` | yes | varies | Ordered list of registered component IDs assigned to this position |

> [!NOTE] The `components` field was renamed from `addons` in an earlier version. The backend DTO still uses the JSON field name `addons` for backwards compatibility, but the frontend always uses `components`. The `migratePanelLayout()` function handles the conversion.

### Default Component Assignments

| Position | Default Components | Default Width/Height |
| -------- | ------------------ | -------------------- |
| left | `['content-navigation']` | 300px |
| right | `['glossary', 'search-results']` | 300px |
| bottom | `[]` (empty) | 200px, hidden |

## `FloatingPanelInstance`

State for a single floating panel window.

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `panelId` | `string` | yes | UUID | Unique instance identifier (generated via `crypto.randomUUID()`) |
| `componentId` | `string` | yes | -- | ID of the registered component being rendered |
| `x` | `number` | yes | `100` | Left offset in pixels from viewport edge |
| `y` | `number` | yes | `100` | Top offset in pixels from viewport edge |
| `width` | `number` | yes | `400` | Panel width in pixels (min: 200) |
| `height` | `number` | yes | `300` | Panel height in pixels (min: 150) |
| `zIndex` | `number` | yes | auto | Stacking order (base: 500, auto-increments on focus) |
| `minimized` | `boolean` | yes | `false` | Whether only the title bar is visible |

## `UIState` (Panel-Related Fields)

Additional panel state stored in the `uiState` section of preferences.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `collapsedPanels.left` | `boolean` | Whether the left sidebar is in mini-panel mode |
| `collapsedPanels.right` | `boolean` | Whether the right sidebar is in mini-panel mode |
| `collapsedPanels.bottom` | `boolean` | Whether the bottom bar is in mini-panel mode |
| `activeTab_left` | `string` | Component ID of the active tab in left position |
| `activeTab_right` | `string` | Component ID of the active tab in right position |
| `activeTab_bottom` | `string` | Component ID of the active tab in bottom position |

## Constraints

- `panelId` is unique across all floating panel instances
- `componentId` must reference a registered component in [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|PanelRegistry]]
- A component can only appear in one docked position at a time (enforced by settings UI)
- A component can exist as both docked and floating simultaneously (no mutual exclusion enforced)
- `zIndex` values are renormalized starting from 500 when any value exceeds 900 (prevents modal z-index collision at 1000+)
- Floating panel dimensions are clamped: `width >= 200`, `height >= 150`
- Floating panel position is clamped to viewport bounds on drag and on window resize

## Backend DTO Mapping

The Go backend uses corresponding DTO structs for serialization:

| Frontend Type | Backend DTO | JSON File |
| ------------- | ----------- | --------- |
| `PanelLayout` | `PanelLayoutDTO` | `preferences.json` |
| `PanelConfig` | `PanelConfigDTO` | (nested) |
| `FloatingPanelInstance` | `FloatingPanelInstanceDTO` | (nested) |

> [!WARNING] The backend `PanelConfigDTO` uses the field name `Addons` (Go) / `addons` (JSON) for what the frontend calls `components`. This is a legacy naming preserved for JSON compatibility. The `migratePanelLayout()` function on the frontend handles the mapping transparently.

## Read By

- [[Palimpsest-Writing-Software/modules/panel-frontend/PanelContainer\|PanelContainer]] -- reads `components` array to determine which tabs to render
- [[Palimpsest-Writing-Software/modules/panel-frontend/MiniPanel\|MiniPanel]] -- reads `components` array for icon strip
- [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanelManager\|FloatingPanelManager]] -- reads `floating` array to render floating panels
- [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanel\|FloatingPanel]] -- reads individual `FloatingPanelInstance` for position/size
- `ResizableLayout` -- reads `width`, `height`, `enabled`, `visible`, `collapsedPanels`
- `UserSettingsModal` -- reads all positions' `components` for the assignment dropdowns

## Written By

- `userPreferences.updatePanelLayout()` -- docked panel size/visibility changes
- `userPreferences.popOutPanel()` -- creates new `FloatingPanelInstance`, removes from docked
- `userPreferences.dockPanel()` -- removes `FloatingPanelInstance`, adds to docked components
- `userPreferences.updateFloatingPanel()` -- position/size updates during drag/resize
- `userPreferences.closeFloatingPanel()` -- removes `FloatingPanelInstance`
- `userPreferences.bringToFront()` -- updates `zIndex`
- `userPreferences.collapsePanel()` / `expandPanel()` -- updates `collapsedPanels`
- `userPreferences.updateUIState()` -- updates `activeTab_*`
- `ensureSearchPanelVisible()` (search store) -- adds search panel to components if missing
