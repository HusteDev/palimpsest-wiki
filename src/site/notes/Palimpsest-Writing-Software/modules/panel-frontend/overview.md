---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/overview/","title":"Panel Frontend Module","tags":["module","panels","frontend"],"updated":"2026-03-07T18:44:54.621-07:00"}
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


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/panel-system/diagrams/panel-rendering-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Panel Rendering: Registry to Screen

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/panel-system/diagrams/pop-out-dock-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Pop-Out and Dock: Floating Panel Lifecycle

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Related

- [[Palimpsest-Writing-Software/features/panel-system/overview\|Feature: Panel System]]
- [[Palimpsest-Writing-Software/data-models/PanelLayout\|Data Model: PanelLayout]]
