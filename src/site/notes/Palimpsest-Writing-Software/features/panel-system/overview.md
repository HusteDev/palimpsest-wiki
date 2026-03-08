---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/panel-system/overview/","title":"Panel System","tags":["feature","panels","core"],"updated":"2026-03-07T18:43:46.973-07:00"}
---


# Panel System

> [!NOTE] A dynamic panel management system supporting docked sidebars (left, right, bottom) with collapsible mini-mode, floating pop-out windows with drag/resize, and a component registry that enables lazy-loaded, position-assignable panel components.

## User-Facing Behavior

The workspace is divided into a central editor area surrounded by up to three docked panel regions (left sidebar, right sidebar, bottom bar). Each region can host one or more panel components as tabs. Users can:

- **Assign panels to positions** via the Settings modal, choosing which components appear in which sidebar
- **Switch tabs** when multiple components share a position
- **Collapse panels** to a narrow icon strip (MiniPanel) that preserves layout space; clicking an icon re-expands
- **Pop out panels** by right-clicking a tab and selecting "Pop Out," creating a floating window overlay
- **Drag floating panels** anywhere within the viewport via the title bar
- **Resize floating panels** from any edge or corner with minimum dimension constraints
- **Minimize floating panels** to title-bar-only mode
- **Dock floating panels** back to left, right, or bottom via the dock menu
- **Close floating panels** which removes them entirely (they do not auto-re-dock)

All panel state (positions, sizes, assignments, collapsed/expanded, floating instances) persists across sessions via the settings API.

## Scope

- **License:** Core
- **Modules involved:**
  - [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]] -- Registry, containers, floating manager, mini panels
- **API endpoints:** Uses [[Palimpsest-Writing-Software/api/glossary/overview\|Settings API]] (`GET/PUT /api/settings`) for persistence -- no dedicated panel endpoints
- **Data models:**
  - [[Palimpsest-Writing-Software/data-models/PanelLayout\|data-models/PanelLayout]] -- PanelLayout, PanelConfig, FloatingPanelInstance

## Architecture

### Component Hierarchy

```
+page.svelte (editor route)
  |-- ResizableLayout.svelte
  |     |-- PanelContainer position="left"    (or MiniPanel if collapsed)
  |     |-- [Central Editor Area]
  |     |-- PanelContainer position="right"   (or MiniPanel if collapsed)
  |     +-- PanelContainer position="bottom"  (or MiniPanel if collapsed)
  +-- FloatingPanelManager.svelte
        +-- FloatingPanel (one per active floating instance)
```

`ResizableLayout` manages the docked panel regions with drag-to-resize dividers. `FloatingPanelManager` sits at the same DOM level (not nested inside the layout) and renders floating panels as `position: fixed` overlays.

### Layer Overview

```
User Interaction
  |
PanelContainer / FloatingPanel / MiniPanel     (rendering + interaction)
  |
PanelRegistry                                   (component metadata + lazy-loading)
  |
userPreferences store                           (state management + mutations)
  |
PUT /api/settings                               (persistence to backend)
  |
preferences.json                                (JSON file on disk)
```

### Registration Model

Panel components register themselves in [[Palimpsest-Writing-Software/modules/panel-frontend/PanelRegistry\|PanelRegistry]] at module load time. Each registration provides:

| Field | Purpose |
| ----- | ------- |
| `id` | Unique identifier (e.g., `'glossary'`, `'search-results'`) |
| `name` | Display name for tabs and settings |
| `icon` | Lucide icon name for tabs and mini-panel |
| `tier` | License tier: `'core'`, `'pro'`, or `'addon'` |
| `alwaysEnabled` | If true, cannot be removed from any position |
| `defaultPosition` | Where the component appears on first launch |
| `allowedPositions` | Which positions the component supports (including `'floating'`) |
| `order` | Sort order within a position's tab bar |
| `component` | Lazy-load function: `() => import('path/to/Component.svelte')` |

### Built-in Panel Components

| ID | Name | Default Position | Allowed Positions | Always Enabled | Tier |
| -- | ---- | ---------------- | ----------------- | -------------- | ---- |
| `content-navigation` | Content Navigator | left | left, right, bottom | yes | core |
| `glossary` | Glossary | right | left, right, bottom, floating | yes | core |
| `search-results` | Search Results | right | left, right, bottom, floating | no | core |

> [!TIP] The content navigator cannot float because it controls editor navigation and is expected to always be accessible in a docked position. Glossary and search results can float because they are reference tools that benefit from side-by-side viewing.

### Key Design Decisions

- **Registry singleton pattern** -- a module-level `Map` ensures components register once at import time; no initialization ceremony needed
- **Lazy-loading via dynamic import** -- panel components are only loaded when their tab becomes active, reducing initial bundle size
- **State ownership in preferences store** -- all panel mutations (pop-out, dock, collapse, resize) flow through `userPreferences`, ensuring a single source of truth
- **Floating panels are peers, not children** -- `FloatingPanelManager` renders at the same DOM level as `ResizableLayout`, not inside it, preventing CSS containment issues with `position: fixed`
- **Z-index normalization** -- floating panels start at z-index 500 and increment; at z-index 900, all indices are renormalized to prevent collision with modals (z-index 1000+)
- **Debounced saves for continuous operations** -- drag and resize use 1000ms debounce to avoid flooding the settings API; discrete actions (pop-out, dock, close) save immediately
- **Floating excluded from settings dropdown** -- users access floating mode via the right-click "Pop Out" context menu, not the settings panel, keeping the settings UI simple
- **Close does not re-dock** -- closing a floating panel removes it entirely; the user must re-assign it from settings if they want it back in a docked position

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


## Integration Points

### Search Auto-Open

The search store's `ensureSearchPanelVisible()` function directly manipulates panel preferences to make the search panel visible and active when a search is performed. If the search panel is not assigned to any docked position, it is auto-added to the right sidebar.

### Glossary Auto-Open

Similarly, `glossaryStore.navigateToEntry()` ensures the glossary panel is visible when navigating to an entry from editor marks or cross-references.

### Theme System

All panel components use CSS custom properties (`--bg-secondary`, `--border`, `--accent`, etc.) and are automatically styled by the active theme. No panel-specific theme overrides exist.

### Editor Page Mount

The editor route (`+page.svelte`) renders `<FloatingPanelManager />` after `</ResizableLayout>` at the top level, ensuring floating panels overlay the entire workspace including all docked panels.

## Security

No security-sensitive operations exist in the panel system. Panel state is stored locally in `preferences.json` and never transmitted beyond the local backend. Panel components themselves handle their own security concerns (e.g., the glossary panel validates input before API calls).

## Performance

- **Lazy-loading** -- panel components are only imported when first activated, keeping initial page load fast. Loaded components are cached in a `Map` to avoid re-importing.
- **Debounced persistence** -- continuous interactions (drag, resize) use 1000ms debounce (`PANEL_DEBOUNCE` from config.ts) to batch settings saves. This prevents flooding the backend during smooth drag operations.
- **Z-index normalization is O(n)** where n is the number of floating panels. Triggered infrequently (only when z-index exceeds 900, which requires ~400 bring-to-front actions without normalization).
- **Window resize clamping is O(n)** -- iterates all floating panels on viewport resize to ensure none escape bounds. Acceptable given typical floating panel counts (<10).
- **Icon resolution** -- Lucide icons are resolved dynamically via `LucideIcons[iconName]`. This is a simple object lookup, not a performance concern.

## 12-Factor Compliance

- **Config via environment:** Panel debounce timing is a code constant (`PANEL_DEBOUNCE = 1000`), not an environment variable. This is acceptable for a UI-only setting with no operational impact.
- **Strict separation:** Panel state flows through the REST API (`PUT /api/settings`) to the backend for persistence. No direct file access from the frontend.
- **Stateless processes:** Each panel rendering cycle reads from the preferences store; no hidden global state exists beyond the registry singleton (which is read-only after startup).
- **Dev/prod parity:** The same panel system runs in both Tauri desktop and development browser modes.

## Logging

All panel components use injected loggers following project conventions:

| Layer | Component | Logger Name |
| ----- | --------- | ----------- |
| Frontend | PanelRegistry | `'panel-registry'` |
| Frontend | PanelContainer | `'panel-container'` |
| Frontend | FloatingPanel | `'floating-panel'` |
| Frontend | FloatingPanelManager | `'floating-panel-manager'` |
| Frontend | MiniPanel | `'mini-panel'` |

## Related

- [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Module: Panel Frontend]]
- [[Palimpsest-Writing-Software/data-models/PanelLayout\|Data Model: PanelLayout]]
