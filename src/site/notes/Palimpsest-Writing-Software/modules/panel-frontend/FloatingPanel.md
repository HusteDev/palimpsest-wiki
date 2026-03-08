---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/floating-panel/","title":"FloatingPanel","tags":["file","panel-frontend"],"updated":"2026-03-05T05:32:43.441-07:00"}
---


# `FloatingPanel.svelte`

**Module:** [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]]
**Path:** `src/lib/components/panels/FloatingPanel.svelte`

> [!NOTE] Individual floating panel window rendered as a `position: fixed` overlay. Supports drag via title bar, 8-direction resize, minimize to title-bar-only, dock back to any docked position, and close. Hidden on mobile viewports.

## Props

| Prop | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `instance` | `FloatingPanelInstance` | yes | Panel state: position, size, z-index, minimized flag |
| `onclose` | `() => void` | yes | Callback when close button clicked |
| `ondock` | `(position: 'left' \| 'right' \| 'bottom') => void` | yes | Callback when docking to a position |
| `onupdate` | `(updates: Partial<FloatingPanelInstance>) => void` | yes | Callback for position/size/minimize changes |
| `onfocus` | `() => void` | yes | Callback when panel receives focus (bring to front) |

## Constants

| Constant | Value | Description |
| -------- | ----- | ----------- |
| `MIN_WIDTH` | `200` | Minimum panel width in pixels |
| `MIN_HEIGHT` | `150` | Minimum panel height in pixels |
| `TITLE_BAR_HEIGHT` | `36` | Height of the title bar in pixels |

## Reactive State

| Variable | Type | Source | Description |
| -------- | ---- | ------ | ----------- |
| `componentConfig` | `PanelComponentConfig` | `getComponentById(instance.componentId)` | Resolved component metadata |
| `LoadedComponent` | `Component \| null` | `$state` | Lazily loaded Svelte component |
| `isLoading` | `boolean` | `$state` | Whether the component is currently loading |
| `loadError` | `string \| null` | `$state` | Error message if loading failed |
| `isDragging` | `boolean` | `$state` | Whether a drag operation is in progress |
| `isResizing` | `boolean` | `$state` | Whether a resize operation is in progress |
| `resizeDirection` | `string` | `$state` | Active resize handle direction (n, s, e, w, ne, nw, se, sw) |
| `showDockMenu` | `boolean` | `$state` | Whether the dock position dropdown is visible |

## Behavior

### Lazy-Loading

On mount, an `$effect` calls `loadComponent()` which invokes `componentConfig.component()` (the dynamic import function from registration). The loaded component is cached in `LoadedComponent` state. Loading and error states are displayed in the content area while the import resolves.

### Drag

Mousedown on the title bar starts a drag operation:

1. Records `dragStartX/Y` (mouse position) and `initialX/Y` (panel position)
2. `handleDrag` calculates delta: `newX = initialX + (mouseX - dragStartX)`
3. Clamps to viewport: `x: [0, window.innerWidth - width]`, `y: [0, window.innerHeight - TITLE_BAR_HEIGHT]`
4. Calls `onupdate({ x, y })` which flows through the preferences store (debounced save)
5. Mouseup ends the drag

### Resize

Eight resize handles are positioned at edges and corners:

| Handle | Cursor | Directions Modified |
| ------ | ------ | ------------------- |
| n | `ns-resize` | top edge, height |
| s | `ns-resize` | bottom edge, height |
| e | `ew-resize` | right edge, width |
| w | `ew-resize` | left edge, width |
| ne | `nesw-resize` | top + right |
| nw | `nwse-resize` | top + left |
| se | `nwse-resize` | bottom + right |
| sw | `nesw-resize` | bottom + left |

All resize operations respect `MIN_WIDTH` and `MIN_HEIGHT` constraints and clamp to viewport bounds. Calls `onupdate({ x, y, width, height })` on each mousemove.

### Minimize

Toggle button in the title bar calls `onupdate({ minimized: !instance.minimized })`. When minimized:

- Content area and resize handles are hidden
- Panel height collapses to title bar only (`height: auto`)
- Only title bar with title, icon, and control buttons remains visible

### Dock Menu

Small dropdown from the title bar with three options: Dock Left, Dock Right, Dock Bottom. Each calls `ondock(position)`. The dropdown closes on selection or when clicking outside.

### Focus (Bring to Front)

Any `mousedown` on the panel element calls `onfocus()`, which triggers `userPreferences.bringToFront(panelId)` -- increments the panel's z-index above all other floating panels.

## Styling

- Uses `position: fixed` with inline `left`, `top`, `width`, `height`, `z-index` from instance state
- Hidden on mobile: `@media (max-width: 768px) { display: none }`
- CSS custom properties for theming: `--bg-secondary`, `--border`, `--text-primary`, `--accent`

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `LucideIcons`, `Minus`, `X`, `PanelLeft`, `PanelRight`, `PanelBottom` | `lucide-svelte` | Icons for title bar and dock menu |
| `Component` | `svelte` | Generic component type |
| `FloatingPanelInstance` | `$lib/stores/preferences` | Instance state type |
| `getComponentById`, `PanelComponentConfig` | `./PanelRegistry` | Component metadata lookup |
| `createLogger` | `$lib/services/loggerService` | Logger (`'floating-panel'`) |

## Side Effects

None directly -- all mutations are delegated to parent via callbacks (`onupdate`, `onclose`, `ondock`, `onfocus`).
