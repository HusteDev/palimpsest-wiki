---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/plugin-loader/","title":"PluginLoader.ts","tags":["file","plugins-frontend"],"updated":"2026-03-05T07:56:21.417-07:00"}
---


# `PluginLoader.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/PluginLoader.ts`

> [!NOTE] Plugin discovery, validation, loading, and lifecycle management service. Discovers plugins from the backend API, validates manifests, dynamically imports frontend modules, and registers content types with the `ContentRendererRegistry`.

## Exports

| Name | Kind | Description |
| --- | --- | --- |
| `pluginStore` | Svelte writable store | Reactive store holding `PluginStoreState`. Provides helper methods: `getPlugin(name)`, `getAllPlugins()`, `getEnabledPlugins()`, `isInitialized()`, `reset()`. Internal mutation methods prefixed with `_` (`_setLoading`, `_setInitialized`, `_setError`, `_setPlugin`, `_removePlugin`) |
| `discoverPlugins` | async function | Fetches `GET /api/plugins` from the backend, returns an array of `DiscoveredPlugin` objects containing `name`, `path`, `source`, and `manifest` |
| `initializePlugins` | async function | Main startup entry point. Calls `discoverPlugins`, then `loadPlugin` for each result. Manages store loading/initialized flags. Continues loading remaining plugins if one fails |
| `enablePlugin` | async function | Enables a loaded plugin by name. Executes the `onEnable` lifecycle hook if present, updates status to `'enabled'`, sends a success notification |
| `disablePlugin` | async function | Disables an enabled plugin by name. Executes the `onDisable` lifecycle hook if present, updates status to `'disabled'`, sends an info notification |
| `getVirtualContentTypes` | function | Returns all registered `VirtualContentType` objects as an array |
| `getVirtualContentType` | function | Returns a single `VirtualContentType` by its full ID (e.g., `"plugin:excalidraw:drawing"`), or `undefined` |
| `isVirtualContentType` | function | Returns `true` if the given content type ID exists in the virtual content types map |
| `cleanup` | function | Unloads all plugins: unregisters all content renderers via `ContentRendererRegistry.unregisterPlugin`, clears the virtual content types map, and resets the plugin store |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| `writable`, `get` | `svelte/store` | Svelte reactive store primitives for `pluginStore` |
| `PluginManifest`, `LoadedPlugin`, `PluginSource`, `PluginStatus`, `PluginStoreState`, `PluginFrontendModule`, `VirtualContentType`, `PluginContext`, `PluginAPI`, `ContentRendererConfig` | `[[modules/plugins-frontend/pluginTypes\|./types]]` | Type imports for the plugin system |
| `validateManifest` | `[[modules/plugins-frontend/manifestSchema\|./manifest-schema]]` | Manifest validation before loading |
| `ContentRendererRegistry` (namespace import) | `[[modules/plugins-frontend/contentRendererRegistry\|./ContentRendererRegistry]]` | Registry for content type renderers |
| `createLogger` | `$lib/services/loggerService` | Structured logging with `plugin-loader` component name |
| `notificationStore` | `$lib/stores/notifications` | User-facing toast notifications for enable/disable events |

## Internal Functions

| Name | Kind | Description |
| --- | --- | --- |
| `loadPlugin` | async function | Loads a single discovered plugin: validates manifest via `validateManifest`, sets status to `'loading'`, calls `loadFrontendModule`, calls `registerContentTypes` if the plugin declares content type capability, then sets status to `'loaded'`. On failure, sets status to `'error'` with message |
| `loadFrontendModule` | async function | Dynamically imports the plugin's frontend bundle from `/api/plugins/{name}/assets/{entryPoint}` using `import()` with `@vite-ignore`. Returns the module cast to `PluginFrontendModule` |
| `registerContentTypes` | function | Iterates over manifest `contentTypes`, builds full IDs via `ContentRendererRegistry.buildPluginContentTypeId`, resolves renderer components from the frontend module, registers each with `ContentRendererRegistry.register`, and creates corresponding `VirtualContentType` entries in the module-level `virtualContentTypes` map. Wires `onContentLoad`/`onContentSave` lifecycle hooks if the module exports a `lifecycle` object |
| `createPluginContext` | function | Creates a `PluginContext` for lifecycle hooks with `pluginId`, `dataDir` (from plugin path), empty `settings` (stub for user preferences), and a `PluginAPI` instance |
| `createPluginAPI` | function | Creates a `PluginAPI` for a specific plugin. `content.get` and `content.update` use `fetch` against `/api/projects/{slug}/content/{path}`. `notifications` delegates to `notificationStore`. `events.on` and `events.dispatch` are stubs that log but do not integrate with `ModuleEventBus` yet. `storage` uses `localStorage` with `plugin:{pluginId}:` prefix namespace |

## Side Effects

- **Module-level state:** Creates a singleton `pluginStore` (Svelte writable store) and a module-level `virtualContentTypes` Map. Both persist for the lifetime of the application.
- **Network requests:** `discoverPlugins` calls `GET /api/plugins`. `loadFrontendModule` performs dynamic `import()` calls to fetch plugin JavaScript bundles. `createPluginAPI.content.get/update` calls REST endpoints.
- **localStorage access:** `createPluginAPI.storage` reads and writes to `localStorage` using namespaced keys (`plugin:{pluginId}:{key}`).
- **Registry mutations:** `registerContentTypes` writes to `ContentRendererRegistry`. `cleanup` removes all entries.
- **Notifications:** `enablePlugin` and `disablePlugin` push toast notifications via `notificationStore`.

## Notes

> [!WARNING] The `createPluginAPI.events` methods (`on` and `dispatch`) are currently stubs. They log the subscription/dispatch but do not connect to the `ModuleEventBus`. This is marked with `TODO` comments in the source.

> [!WARNING] `createPluginContext` returns an empty `settings` object. Loading persisted user preferences is marked as a `TODO`.

> [!WARNING] `initializePlugins` is guarded against double-initialization. If `pluginStore.isInitialized()` returns `true`, the function returns immediately without re-discovering plugins.

> [!WARNING] Plugin loading failures are non-fatal. If one plugin fails to load, the others continue. The failed plugin's status is set to `'error'` with a descriptive message.
