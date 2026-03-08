---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/content-renderer-registry/","title":"ContentRendererRegistry.ts","tags":["file","plugins-frontend"],"updated":"2026-03-05T07:56:47.414-07:00"}
---


# `ContentRendererRegistry.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/ContentRendererRegistry.ts`

> [!NOTE] Central registry for plugin content renderers. Plugins register custom Svelte components that replace the default ProseMirror TextEditor for their content types.

## Exports

| Name | Kind | Description |
| --- | --- | --- |
| `register` | function | Registers a `ContentRendererConfig` for a content type ID. Logs a warning and skips if a renderer is already registered for that ID |
| `unregister` | function | Removes a renderer by content type ID. Also clears the component cache entry. Returns `true` if the entry existed |
| `unregisterPlugin` | function | Removes all renderers belonging to a specific plugin ID. Returns the count of renderers removed |
| `hasCustomRenderer` | function | Returns `true` if a custom renderer is registered for the given content type ID. Used by the editor page to decide between TextEditor and a plugin renderer |
| `getRenderer` | function | Returns the `ContentRendererConfig` for a content type ID, or `undefined` if not found |
| `getAllRenderers` | function | Returns an array of all registered `ContentRendererConfig` objects |
| `getRenderersForPlugin` | function | Returns an array of `ContentRendererConfig` objects filtered to a specific plugin ID |
| `loadComponent` | async function | Lazily loads and caches the Svelte `Component` for a content type ID. Checks `componentCache` first, then invokes the `config.component()` factory, caches the result, and returns it. Throws if no renderer is registered or loading fails |
| `executeOnLoad` | async function | Executes the `onLoad` lifecycle hook for a content type if defined. Falls back to returning `content.content` if no hook is registered or if the hook throws |
| `executeOnSave` | async function | Executes the `onSave` lifecycle hook for a content type if defined. Falls back to returning the original `data` if no hook is registered or if the hook throws |
| `clearRegistry` | function | Clears both the `registry` and `componentCache` Maps. Primarily for testing |
| `size` | function | Returns the number of registered renderers (`registry.size`) |
| `isPluginContentType` | function | Returns `true` if a content type ID starts with `"plugin:"` |
| `parsePluginContentTypeId` | function | Parses a plugin content type ID (format `"plugin:{pluginId}:{typeId}"`) into `{ pluginId, typeId }`, or returns `null` if the format is invalid |
| `buildPluginContentTypeId` | function | Constructs a full content type ID string from a plugin ID and type ID: `"plugin:{pluginId}:{typeId}"` |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| `Component` | `svelte` | Type reference for cached Svelte components |
| `Content` | `$lib/types/content` | Type reference for content items passed to lifecycle hooks |
| `ContentRendererConfig` | `[[modules/plugins-frontend/pluginTypes\|./types]]` | Configuration interface for registered renderers |
| `createLogger` | `$lib/services/loggerService` | Structured logging with `content-renderer-registry` component name |

## Side Effects

- **Module-level state:** Two module-scoped `Map` instances persist for the application lifetime:
  - `registry`: `Map<string, ContentRendererConfig>` -- maps content type IDs to their renderer configurations.
  - `componentCache`: `Map<string, Component>` -- caches dynamically loaded Svelte components to avoid redundant imports.
- **Dynamic imports:** `loadComponent` triggers dynamic `import()` calls via the `config.component()` factory when a renderer component is first requested.

## Notes

> [!WARNING] `register` silently skips duplicate registrations rather than throwing. If two plugins attempt to register the same content type ID, only the first registration takes effect and a warning is logged.

> [!WARNING] `executeOnLoad` and `executeOnSave` are fault-tolerant. If a lifecycle hook throws an error, the original data is returned and the error is logged. This prevents plugin bugs from corrupting content save/load operations.

> [!WARNING] Plugin content type IDs follow a strict three-segment format: `plugin:{pluginId}:{typeId}`. The `parsePluginContentTypeId` function returns `null` for IDs that do not have exactly three colon-separated segments.
