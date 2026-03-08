---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/plugin-index/","title":"index.ts","tags":["file","plugins-frontend"],"updated":"2026-03-05T07:58:59.398-07:00"}
---


# `index.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/index.ts`

> [!NOTE] Barrel export file for the plugin system. Provides a clean public API surface by re-exporting selected items from all internal modules, with some renamed for clarity.

## Exports

### From `[[modules/plugins-frontend/pluginTypes|types.ts]]` (type re-exports)

| Name | Kind | Description |
| --- | --- | --- |
| `PluginManifest` | type | Complete plugin manifest interface |
| `PluginAuthor` | type | Author metadata |
| `PluginCapabilities` | type | Capability flags |
| `PluginPermissions` | type | Permission declarations |
| `PluginEntryPoints` | type | Code entry points |
| `PluginContentTypeDefinition` | type | Content type definition in manifest |
| `PluginPanelDefinition` | type | Panel definition in manifest |
| `PluginContextMenuItemDefinition` | type | Context menu item definition in manifest |
| `PluginJobHandlerDefinition` | type | Job handler definition (Pro only) |
| `PluginRestEndpointDefinition` | type | REST endpoint definition (Pro only) |
| `PluginTier` | type | `'core' \| 'pro'` union |
| `PluginSource` | type | Plugin load origin |
| `PluginStatus` | type | Runtime status |
| `LoadedPlugin` | type | Loaded plugin instance |
| `PluginFrontendModule` | type | Frontend module shape |
| `PluginContext` | type | Lifecycle hook context |
| `PluginAPI` | type | Plugin API surface |
| `PluginLifecycle` | type | Lifecycle hooks interface |
| `CreatePluginContentInput` | type | Content creation input |
| `PluginRendererProps` | type | Renderer component props |
| `ContentRendererConfig` | type | Renderer registration config |
| `ContextMenuContext` | type | Context menu handler context |
| `PluginContextMenuHandler` | type | Context menu handler signature |
| `VirtualContentType` | type | Frontend-only content type |
| `PluginStoreState` | type | Plugin store state |
| `PluginUserSettings` | type | Per-plugin user settings |
| `PluginSettings` | type | All plugin settings |

### From `[[modules/plugins-frontend/manifestSchema|manifest-schema.ts]]`

| Name | Kind | Description |
| --- | --- | --- |
| `validateManifest` | function | Manifest validation |
| `pluginManifestSchema` | const | JSON Schema Draft 7 definition |
| `ManifestValidationResult` | type | Validation result interface |
| `ManifestValidationError` | type | Validation error interface |

### From `[[modules/plugins-frontend/contentRendererRegistry|ContentRendererRegistry.ts]]` (with aliases)

| Export Name | Original Name | Kind | Description |
| --- | --- | --- | --- |
| `registerRenderer` | `register` | function | Register a content renderer |
| `unregisterRenderer` | `unregister` | function | Unregister by content type ID |
| `unregisterPluginRenderers` | `unregisterPlugin` | function | Unregister all renderers for a plugin |
| `hasCustomRenderer` | `hasCustomRenderer` | function | Check if a custom renderer exists |
| `getRenderer` | `getRenderer` | function | Get renderer config by content type ID |
| `getAllRenderers` | `getAllRenderers` | function | Get all registered renderer configs |
| `getRenderersForPlugin` | `getRenderersForPlugin` | function | Get renderers for a specific plugin |
| `loadRendererComponent` | `loadComponent` | function | Lazy-load and cache a renderer component |
| `executeOnLoad` | `executeOnLoad` | function | Execute onLoad lifecycle hook |
| `executeOnSave` | `executeOnSave` | function | Execute onSave lifecycle hook |
| `clearRendererRegistry` | `clearRegistry` | function | Clear all renderer registrations |
| `rendererCount` | `size` | function | Get count of registered renderers |
| `isPluginContentType` | `isPluginContentType` | function | Check if ID starts with `"plugin:"` |
| `parsePluginContentTypeId` | `parsePluginContentTypeId` | function | Parse a plugin content type ID |
| `buildPluginContentTypeId` | `buildPluginContentTypeId` | function | Build a plugin content type ID |

### From `[[modules/plugins-frontend/pluginLoader|PluginLoader.ts]]` (with alias)

| Export Name | Original Name | Kind | Description |
| --- | --- | --- | --- |
| `pluginStore` | `pluginStore` | store | Svelte writable store for plugin state |
| `discoverPlugins` | `discoverPlugins` | function | Fetch plugins from backend |
| `initializePlugins` | `initializePlugins` | function | Main startup entry point |
| `enablePlugin` | `enablePlugin` | function | Enable a plugin |
| `disablePlugin` | `disablePlugin` | function | Disable a plugin |
| `getVirtualContentTypes` | `getVirtualContentTypes` | function | Get all virtual content types |
| `getVirtualContentType` | `getVirtualContentType` | function | Get virtual content type by ID |
| `isVirtualContentType` | `isVirtualContentType` | function | Check if ID is a virtual type |
| `cleanupPlugins` | `cleanup` | function | Unload all plugins and reset |

### From `[[modules/plugins-frontend/security|security.ts]]`

| Name | Kind | Description |
| --- | --- | --- |
| `checkPermission` | function | Check a specific permission |
| `summarizePermissions` | function | Generate UI permission summaries |
| `validatePluginSecurity` | function | Security validation of manifest |
| `logPermissionRequest` | function | Record permission request to audit log |
| `getPermissionLog` | function | Retrieve audit log entries |
| `clearPermissionLog` | function | Clear the audit log |
| `PermissionCheckResult` | type | Permission check result |
| `PermissionType` | type | Permission type union |
| `PermissionSummary` | type | Permission summary for UI |
| `SecurityValidationResult` | type | Security validation result |

### From `[[modules/plugins-frontend/sandbox|sandbox.ts]]`

| Name | Kind | Description |
| --- | --- | --- |
| `createSandboxedAPI` | function | Main factory for complete sandboxed API |
| `createSandboxedFetch` | function | Factory for permission-checked fetch |
| `createSandboxedStorage` | function | Factory for namespaced localStorage |
| `createSandboxedClipboard` | function | Factory for permission-checked clipboard |
| `PluginSandboxError` | class | Base sandbox error |
| `PluginPermissionError` | class | Permission denied error |
| `PluginTimeoutError` | class | Request timeout error |
| `PluginStorageError` | class | Storage operation error |
| `PluginClipboardError` | class | Clipboard operation error |
| `SandboxedAPI` | type | Complete sandboxed API interface |
| `SandboxedFetchOptions` | type | Fetch options with timeout |
| `SandboxedStorage` | type | Namespaced storage interface |
| `SandboxedClipboard` | type | Clipboard interface |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| `./types` | `[[modules/plugins-frontend/pluginTypes\|types.ts]]` | All type definitions |
| `./manifest-schema` | `[[modules/plugins-frontend/manifestSchema\|manifest-schema.ts]]` | Schema and validation |
| `./ContentRendererRegistry` | `[[modules/plugins-frontend/contentRendererRegistry\|ContentRendererRegistry.ts]]` | Renderer registry functions |
| `./PluginLoader` | `[[modules/plugins-frontend/pluginLoader\|PluginLoader.ts]]` | Plugin discovery and loading |
| `./security` | `[[modules/plugins-frontend/security\|security.ts]]` | Permission checking and audit |
| `./sandbox` | `[[modules/plugins-frontend/sandbox\|sandbox.ts]]` | Sandboxed API factories and errors |

## Side Effects

None. This file only re-exports from other modules and does not execute any code at import time beyond what the re-exported modules themselves execute.

## Notes

> [!WARNING] Several exports are renamed from their source module names to provide a more descriptive public API. Notable renames: `register` becomes `registerRenderer`, `loadComponent` becomes `loadRendererComponent`, `cleanup` becomes `cleanupPlugins`, `clearRegistry` becomes `clearRendererRegistry`, `size` becomes `rendererCount`. Always use the barrel export names when importing from `$lib/plugins`.

> [!WARNING] Internal functions (prefixed with `_` in `pluginStore`, and `loadPlugin`/`loadFrontendModule`/`registerContentTypes`/`createPluginContext`/`createPluginAPI` in `PluginLoader.ts`) are intentionally not re-exported. They are implementation details not part of the public API.
