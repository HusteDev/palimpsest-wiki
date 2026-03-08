---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/mini-panel/","title":"MiniPanel","tags":["file","panel-frontend"],"updated":"2026-03-05T05:32:43.651-07:00"}
---


# `MiniPanel.svelte`

**Module:** [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]]
**Path:** `src/lib/components/panels/MiniPanel.svelte`

> [!NOTE] Collapsed panel view showing a vertical (or horizontal for bottom) strip of icons. Displayed when a docked panel position is collapsed. Clicking an icon expands the panel and activates that component's tab.

## Props

| Prop | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `position` | `DockedPanelPosition` | yes | Which collapsed position this represents |

## Reactive State

| Variable | Type | Source | Description |
| -------- | ---- | ------ | ----------- |
| `assignedIds` | `string[]` | `$userPreferences.panelLayout[position].components` | Component IDs assigned to this position |
| `assignedComponents` | `PanelComponentConfig[]` | `getComponentsForIds(assignedIds)` | Resolved and sorted component configs |
| `hasComponents` | `boolean` | derived | Whether any components are assigned |

## Behavior

### Rendering

Displays a column of icon buttons, one per assigned component. Each button shows the component's icon (resolved via `LucideIcons[config.icon]`). If the icon is not found in the Lucide library, falls back to displaying the first letter of the component's name as text.

For the bottom position, icons are arranged horizontally instead of vertically.

### Click-to-Expand

Clicking any icon button calls:

```ts
userPreferences.expandPanel(position, componentId)
```

This performs two actions:
1. Sets `uiState.collapsedPanels[position] = false` (un-collapses)
2. Sets `uiState['activeTab_' + position] = componentId` (activates the clicked component's tab)

`ResizableLayout` detects the `collapsedPanels` change and switches from rendering `MiniPanel` back to [[Palimpsest-Writing-Software/modules/panel-frontend/PanelContainer\|PanelContainer]], which opens with the selected tab active.

### Empty State

If `hasComponents` is false (no components assigned to this position), the mini-panel renders nothing.

## Decision Points

| Condition | Branch |
| --------- | ------ |
| `!hasComponents` | Renders nothing |
| `position === 'bottom'` | Horizontal icon layout |
| `position !== 'bottom'` | Vertical icon layout |
| `LucideIcons[config.icon]` exists | Renders Lucide icon |
| Icon not found | Renders first letter of component name |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `LucideIcons` | `lucide-svelte` | Dynamic icon resolution |
| `Component` | `svelte` | Type reference |
| `userPreferences` | `$lib/stores/preferences` | Store for expand action and panel layout |
| `getComponentsForIds`, `DockedPanelPosition` | `./PanelRegistry` | Registry lookup and types |
| `createLogger` | `$lib/services/loggerService` | Logger (`'mini-panel'`) |

## Side Effects

- `expandPanel()` modifies `uiState.collapsedPanels` and `activeTab_*` (immediate save)
