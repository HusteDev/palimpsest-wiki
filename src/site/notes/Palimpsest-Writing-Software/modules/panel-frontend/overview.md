---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/overview/","title":"Panel Frontend Module","tags":["module","panels","frontend"]}
---


# Panel Frontend Module

> [!NOTE] Svelte 5 frontend module providing the panel registry, docked panel containers with tab management, collapsible mini-panels, floating panel windows with drag/resize, and the floating panel orchestrator.

## Responsibilities

- Component registration and metadata lookup via singleton registry
- Docked panel rendering with tab bar, lazy-loading, and collapse/expand
- Floating panel rendering with drag, resize, minimize, dock, and close
- Orchestration of all active floating panel instances
- Collapsed mini-panel icon strip with expand-on-click
- Barrel exports for public API surface

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|register()]] | function | Register a panel component with metadata |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|getComponentById()]] | function | Lookup a registered component by ID |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|getRegisteredComponents()]] | function | Get all registered component configs |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|getComponentsForIds()]] | function | Resolve ID array to sorted configs |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|getDefaultLayout()]] | function | Build default docked layout from registrations |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|isAllowedAtPosition()]] | function | Check if a component supports a position |
| [[Palimpsest-Writing-Software/modules/panel-frontend/PanelContainer\|PanelContainer]] | component | Docked panel container with tabs |
| [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanel\|FloatingPanel]] | component | Individual floating panel window |
| [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanelManager\|FloatingPanelManager]] | component | Orchestrator for all floating panels |
| [[Palimpsest-Writing-Software/modules/panel-frontend/MiniPanel\|MiniPanel]] | component | Collapsed panel icon strip |
| `PanelComponentConfig` | type | Registration config interface |
| `PanelPosition` | type | `'left' \| 'right' \| 'bottom' \| 'floating'` |
| `PanelTier` | type | `'core' \| 'pro' \| 'addon'` |
| `PanelSource` | type | `'builtin' \| 'plugin'` |

## Internal Structure

```
src/lib/components/panels/
  |-- index.ts                    (barrel exports)
  |-- PanelRegistry.ts            (singleton registry + built-in registrations)
  |-- PanelContainer.svelte       (docked panel with tabs + lazy-loading)
  |-- FloatingPanel.svelte        (floating window with drag/resize)
  |-- FloatingPanelManager.svelte (orchestrator for floating instances)
  +-- MiniPanel.svelte            (collapsed icon strip)
```

### Component Relationships

- **PanelContainer** reads component IDs from `userPreferences.panelLayout[position].components`, resolves them via [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|PanelRegistry]], and lazy-loads the active tab's Svelte component
- **MiniPanel** reads the same component IDs and renders icon buttons; clicking one calls `userPreferences.expandPanel()`
- **FloatingPanelManager** reads `userPreferences.panelLayout.floating[]` and renders a [[Palimpsest-Writing-Software/modules/panel-frontend/FloatingPanel\|FloatingPanel]] for each instance
- **FloatingPanel** resolves its `componentId` via the registry, lazy-loads the component, and delegates all mutations (drag, resize, dock, close) to the preferences store via callbacks
- **ResizableLayout** (in `workspace/`) decides whether to render PanelContainer or MiniPanel based on `uiState.collapsedPanels[position]`

## Dependencies

| Dependency | Type | Purpose |
| ---------- | ---- | ------- |
| `userPreferences` store | internal | Panel layout state, mutations, persistence |
| `LucideIcons` | external | Dynamic icon resolution for tabs and mini-panels |
| `svelte` Component type | external | Generic component reference for lazy-loading |
| `createLogger` | internal | Logger factory for debug/info output |

## Dataflow Diagrams

- [[Palimpsest-Writing-Software/features/panel-system/diagrams/panel-rendering-flow\|Panel Rendering: Registry to Screen]]
- [[Palimpsest-Writing-Software/features/panel-system/diagrams/pop-out-dock-flow\|Pop-Out and Dock: Floating Panel Lifecycle]]

## Related

- [[Palimpsest-Writing-Software/features/panel-system/overview\|Feature: Panel System]]
- [[Palimpsest-Writing-Software/data-models/PanelLayout\|Data Model: PanelLayout]]
