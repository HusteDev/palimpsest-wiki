---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/floating-panel-manager/","title":"FloatingPanelManager","tags":["file","panel-frontend"]}
---


# `FloatingPanelManager.svelte`

**Module:** [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]]
**Path:** `src/lib/components/panels/FloatingPanelManager.svelte`

> [!NOTE] Orchestrator component that renders all active floating panel instances. Mounted at the top level of the editor page (sibling to ResizableLayout, not inside it) to ensure floating panels overlay the entire workspace.

## Props

None. This component reads directly from the `userPreferences` store.

## Reactive State

| Variable | Type | Source | Description |
| -------- | ---- | ------ | ----------- |
| `floatingPanels` | `FloatingPanelInstance[]` | `$userPreferences.panelLayout.floating ?? []` | All active floating panel instances |

## Behavior

### Rendering

Iterates `floatingPanels` with `{#each floatingPanels as panel (panel.panelId)}`, rendering a [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanel\|FloatingPanel]] component for each instance. The `panelId` key ensures Svelte correctly tracks identity across additions and removals.

### Event Delegation

Each FloatingPanel's callback props are wired to the preferences store:

| Callback | Store Action | Save Mode |
| -------- | ------------ | --------- |
| `onclose` | `userPreferences.closeFloatingPanel(panelId)` | immediate |
| `ondock(position)` | `userPreferences.dockPanel(panelId, position)` | immediate |
| `onupdate(updates)` | `userPreferences.updateFloatingPanel(panelId, updates)` | debounced (1000ms) |
| `onfocus` | `userPreferences.bringToFront(panelId)` | debounced |

### Window Resize Clamping

Listens for `svelte:window on:resize` events. When the browser window is resized, iterates all floating panels and clamps any that would extend beyond the new viewport bounds:

```
maxX = window.innerWidth - panel.width
maxY = window.innerHeight - 36  (TITLE_BAR_HEIGHT)
```

If a panel's `x` or `y` exceeds the new maximum, calls `userPreferences.updateFloatingPanel(panelId, { x: clampedX, y: clampedY })`.

## Mount Point

This component must be rendered **after** `ResizableLayout` in the editor page DOM, not inside it. This is critical because:

1. `FloatingPanel` uses `position: fixed` which is relative to the viewport
2. If placed inside a parent with `transform`, `filter`, or `will-change`, `position: fixed` would be relative to that parent instead
3. Rendering at the top level ensures floating panels reliably overlay the entire workspace

```svelte
<!-- In +page.svelte -->
<ResizableLayout>
    {#snippet children()}...{/snippet}
</ResizableLayout>
<FloatingPanelManager />  <!-- Sibling, not child -->
```

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `userPreferences` | `$lib/stores/preferences` | Store for floating panel state and mutations |
| `FloatingPanelInstance` | `$lib/stores/preferences` | Type for panel instance data |
| `FloatingPanel` | `./FloatingPanel.svelte` | Individual floating panel component |
| `createLogger` | `$lib/services/loggerService` | Logger (`'floating-panel-manager'`) |

## Side Effects

- Window resize listener modifies floating panel positions via `updateFloatingPanel` (debounced)
