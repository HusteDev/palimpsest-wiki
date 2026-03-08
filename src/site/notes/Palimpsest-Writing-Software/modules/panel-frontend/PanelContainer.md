---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/panel-container/","title":"PanelContainer","tags":["file","panel-frontend"],"updated":"2026-03-05T05:32:43.860-07:00"}
---


# `PanelContainer.svelte`

**Module:** [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]]
**Path:** `src/lib/components/panels/PanelContainer.svelte`

> [!NOTE] Generic docked panel container that renders registered components as tabs. One instance exists per docked position (left, right, bottom), created by ResizableLayout. Handles tab switching, lazy-loading, collapse, and the "Pop Out" context menu.

## Props

| Prop | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `position` | `DockedPanelPosition` | yes | Which docked slot this container occupies (`'left'`, `'right'`, or `'bottom'`) |

## Reactive State

| Variable | Type | Source | Description |
| -------- | ---- | ------ | ----------- |
| `assignedIds` | `string[]` | `$userPreferences.panelLayout[position].components` | Component IDs assigned to this position |
| `assignedComponents` | `PanelComponentConfig[]` | `getComponentsForIds(assignedIds)` | Resolved and sorted configs |
| `activeTabId` | `string` | `$state`, persisted in `uiState` | Currently active tab's component ID |
| `loadedComponents` | `Map<string, Component>` | `$state` | Cache of lazily loaded Svelte components |
| `loadingIds` | `Set<string>` | `$state` | Component IDs currently being loaded |
| `errorIds` | `Map<string, string>` | `$state` | Component IDs that failed to load, with error messages |
| `contextMenu` | `{ x, y, componentId } \| null` | `$state` | Right-click context menu state |

## Behavior

### Tab Management

If multiple components are assigned to a position, a tab bar renders at the top with one tab per component. Each tab shows the component's icon (resolved via `LucideIcons[config.icon]`) and name. Single-component positions show the title without a tab bar. The active tab is persisted across sessions via `userPreferences.updateUIState({ ['activeTab_' + position]: id })`.

### Lazy-Loading

Components are loaded via dynamic `import()` only when their tab first becomes active. The loading sequence:

1. Tab activated -> check `loadedComponents` cache
2. If cached, render immediately
3. If not cached, add to `loadingIds` set, show spinner
4. Call `config.component()` (the lazy-load function from registration)
5. On success: add to `loadedComponents` cache, remove from `loadingIds`
6. On error: add to `errorIds` with error message, remove from `loadingIds`, show retry button

### Collapse

The collapse button (position-specific icon: `PanelLeftClose`, `PanelRightClose`, or `PanelBottomClose`) calls `userPreferences.collapsePanel(position)`. This sets `uiState.collapsedPanels[position] = true`. `ResizableLayout` detects this and switches from rendering `PanelContainer` to [[Palimpsest-Writing-Software/modules/panel-frontend/MiniPanel\|MiniPanel]].

### Pop-Out Context Menu

Right-clicking a tab opens a context menu with "Pop Out" option, but only if the component's `allowedPositions` includes `'floating'`. Clicking "Pop Out" calls `userPreferences.popOutPanel(componentId, position, { x, y })`, which:

1. Removes the component from this position's `components` array
2. Creates a new `FloatingPanelInstance` with the component
3. The [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanelManager\|FloatingPanelManager]] reactively picks up the new instance

The context menu closes on any click outside or on Escape key.

## Decision Points

| Condition | Branch |
| --------- | ------ |
| `assignedComponents.length === 0` | Renders nothing (empty position) |
| `assignedComponents.length === 1` | Shows title bar without tab strip |
| `assignedComponents.length > 1` | Shows full tab bar |
| `loadedComponents.has(activeTabId)` | Renders cached component |
| `loadingIds.has(activeTabId)` | Shows loading spinner |
| `errorIds.has(activeTabId)` | Shows error message with retry button |
| `!isAllowedAtPosition(id, 'floating')` | Hides "Pop Out" from context menu |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `LucideIcons` | `lucide-svelte` | Dynamic icon resolution for tabs |
| `ExternalLink` | `lucide-svelte` | Pop-out context menu icon |
| `PanelLeftClose`, `PanelRightClose`, `PanelBottomClose` | `lucide-svelte` | Position-specific collapse icons |
| `Component` | `svelte` | Generic component type for rendering |
| `userPreferences` | `$lib/stores/preferences` | State store for layout and UI state |
| `getComponentsForIds`, `PanelComponentConfig`, `PanelPosition`, `DockedPanelPosition` | `./PanelRegistry` | Registry lookups and types |
| `createLogger` | `$lib/services/loggerService` | Logger (`'panel-container'`) |

## Side Effects

- Persists active tab to `uiState` on tab change (debounced via `updateUIState`)
- Collapse action modifies `uiState.collapsedPanels` (immediate save)
- Pop-out action modifies `panelLayout` (immediate save) -- removes from docked, creates floating
