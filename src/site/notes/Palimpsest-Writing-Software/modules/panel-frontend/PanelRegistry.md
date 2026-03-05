---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/panel-frontend/panel-registry/","title":"PanelRegistry","tags":["file","panel-frontend"]}
---


# `PanelRegistry.ts`

**Module:** [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]]
**Path:** `src/lib/components/panels/PanelRegistry.ts`

> [!NOTE] Singleton registry for panel components. All panel components register here at module load time. Provides lookup functions used by PanelContainer, FloatingPanel, MiniPanel, and UserSettingsModal to resolve component metadata and lazy-load functions.

## Types

### `PanelPosition`

```ts
type PanelPosition = 'left' | 'right' | 'bottom' | 'floating'
```

All possible positions a panel can occupy. Docked positions are `left`, `right`, `bottom`. The `floating` position indicates a pop-out window.

### `DockedPanelPosition`

```ts
type DockedPanelPosition = 'left' | 'right' | 'bottom'
```

Subset of `PanelPosition` excluding `floating`. Used by components that only deal with docked panels.

### `PanelTier`

```ts
type PanelTier = 'core' | 'pro' | 'addon'
```

License tier that a panel component requires.

### `PanelSource`

```ts
type PanelSource = 'builtin' | 'plugin'
```

Whether the component ships with the app or comes from a third-party plugin.

### `PanelComponentConfig`

```ts
interface PanelComponentConfig {
    id: string
    name: string
    icon: string
    tier: PanelTier
    alwaysEnabled: boolean
    defaultPosition: DockedPanelPosition
    allowedPositions: PanelPosition[]
    defaultEnabled: boolean
    order: number
    source: PanelSource
    component: () => Promise<{ default: Component }>
}
```

Full configuration for a registered panel component. The `component` field is a lazy-load function that returns a dynamic import promise.

### `DefaultPanelLayout`

```ts
interface DefaultPanelLayout {
    components: string[]
}
```

Used by `getDefaultLayout()` to build the initial layout from registered defaults.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `register` | function | Register a panel component |
| `getComponentById` | function | Lookup by ID |
| `getRegisteredComponents` | function | Get all registrations |
| `getComponentsForIds` | function | Resolve ID array to sorted configs |
| `isAllowedAtPosition` | function | Check position compatibility |
| `getDefaultLayout` | function | Build default layout from registrations |
| `clearRegistry` | function | Testing only -- clears all registrations |
| `PanelComponentConfig` | type | Config interface |
| `PanelPosition` | type | Position union type |
| `DockedPanelPosition` | type | Docked-only position type |
| `PanelTier` | type | License tier type |
| `PanelSource` | type | Component source type |
| `DefaultPanelLayout` | type | Default layout shape |

## Functions

### `register(config)`

```ts
function register(config: PanelComponentConfig): void
```

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `config` | `PanelComponentConfig` | yes | Full component configuration |

Adds a component to the registry `Map`. Logs a warning and skips if a component with the same `id` is already registered (prevents duplicate registrations from hot-reload or multiple imports).

### `getComponentById(id)`

```ts
function getComponentById(id: string): PanelComponentConfig | undefined
```

Returns the config for a given component ID, or `undefined` if not registered.

### `getRegisteredComponents()`

```ts
function getRegisteredComponents(): PanelComponentConfig[]
```

Returns all registered configs as an array. Used by `UserSettingsModal` to populate the panel assignment dropdowns.

### `getComponentsForIds(componentIds)`

```ts
function getComponentsForIds(componentIds: string[]): PanelComponentConfig[]
```

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `componentIds` | `string[]` | yes | Array of component IDs to resolve |

Resolves an array of component IDs to their configs. Filters out any IDs that are not registered (handles uninstalled plugins gracefully). Sorts results by `order` field. This is the primary lookup used by [[Palimpsest-Writing-Software/modules/panel-frontend/PanelContainer\|PanelContainer]] and [[Palimpsest-Writing-Software/modules/panel-frontend/MiniPanel\|MiniPanel]].

### `isAllowedAtPosition(componentId, position)`

```ts
function isAllowedAtPosition(componentId: string, position: PanelPosition): boolean
```

Returns `true` if the component's `allowedPositions` includes the given position. Returns `false` if the component is not registered.

### `getDefaultLayout()`

```ts
function getDefaultLayout(): Record<DockedPanelPosition, DefaultPanelLayout>
```

Builds the default panel layout by iterating all registered components and grouping them by their `defaultPosition`. Only includes components with `defaultEnabled: true`. Explicitly excludes any component whose `defaultPosition` is `'floating'` (floating panels only appear via user action). Returns an object with `left`, `right`, `bottom` keys, each containing a `components` string array.

### `clearRegistry()`

```ts
function clearRegistry(): void
```

Clears all registrations. Intended for testing only -- allows a clean slate between test cases.

## Built-in Registrations

The bottom of the file registers three built-in panel components:

```ts
register({
    id: 'content-navigation', name: 'Content Navigator', icon: 'FolderTree',
    tier: 'core', alwaysEnabled: true, defaultPosition: 'left',
    allowedPositions: ['left', 'right', 'bottom'],
    defaultEnabled: true, order: 0, source: 'builtin',
    component: () => import('$lib/components/content/ContentTree.svelte')
});

register({
    id: 'glossary', name: 'Glossary', icon: 'BookOpen',
    tier: 'core', alwaysEnabled: true, defaultPosition: 'right',
    allowedPositions: ['left', 'right', 'bottom', 'floating'],
    defaultEnabled: true, order: 5, source: 'builtin',
    component: () => import('$lib/components/glossary/GlossaryPanel.svelte')
});

register({
    id: 'search-results', name: 'Search Results', icon: 'Search',
    tier: 'core', alwaysEnabled: false, defaultPosition: 'right',
    allowedPositions: ['left', 'right', 'bottom', 'floating'],
    defaultEnabled: false, order: 10, source: 'builtin',
    component: () => import('$lib/components/search/SearchResults.svelte')
});
```

> [!TIP] New panel components (including future Pro and addon panels) register by calling `register()` with their config. The lazy-load function pattern means the component's code is never loaded until the user activates it.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Component` | `svelte` | Type for the lazy-loaded component reference |
| `createLogger` | `$lib/services/loggerService` | Logger for registration warnings |

## Side Effects

Module-level side effect: the three built-in `register()` calls at the bottom execute when this module is first imported. Since `PanelRegistry.ts` is imported by `PanelContainer`, `MiniPanel`, `FloatingPanel`, `UserSettingsModal`, and the barrel `index.ts`, these registrations happen early in app startup.
