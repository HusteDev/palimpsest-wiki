---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/overview/","title":"Plugins Frontend Module","tags":["module","plugins-frontend"],"updated":"2026-03-05T07:53:30.431-07:00"}
---


# Plugins Frontend

> [!NOTE] Svelte frontend plugin runtime providing manifest validation, plugin discovery via backend API, dynamic module loading, content renderer registry, permission-gated sandbox APIs, security validation with audit logging, and a reactive Svelte store for plugin state management.

## Responsibilities

- Validate plugin manifests against JSON Schema rules on the client side
- Discover plugins by fetching from the backend `GET /api/plugins` endpoint
- Load plugin frontend modules via dynamic `import()` from the backend asset server
- Register and manage custom content renderers that replace the default ProseMirror editor
- Manage virtual content types (frontend-only, no DB migration) in a type registry
- Provide permission-checked sandboxed APIs for fetch, storage, and clipboard
- Track and audit all permission check results with bounded in-memory logging
- Validate plugin security posture and generate human-readable permission summaries
- Manage plugin lifecycle (enable, disable) with hook execution
- Provide a reactive Svelte store for plugin state across the UI

## Public API Surface

| Export | Kind | Source | Description |
| ------ | ---- | ------ | ----------- |
| `[[modules/plugins-frontend/pluginLoader\|pluginStore]]` | store | PluginLoader | Svelte writable store for plugin state |
| `[[modules/plugins-frontend/pluginLoader\|discoverPlugins]]` | function | PluginLoader | Fetch plugin list from backend API |
| `[[modules/plugins-frontend/pluginLoader\|initializePlugins]]` | function | PluginLoader | Main startup: discover → load → register |
| `[[modules/plugins-frontend/pluginLoader\|enablePlugin]]` | function | PluginLoader | Enable a loaded plugin with lifecycle hook |
| `[[modules/plugins-frontend/pluginLoader\|disablePlugin]]` | function | PluginLoader | Disable an enabled plugin with lifecycle hook |
| `[[modules/plugins-frontend/pluginLoader\|getVirtualContentTypes]]` | function | PluginLoader | Get all virtual content types |
| `[[modules/plugins-frontend/pluginLoader\|getVirtualContentType]]` | function | PluginLoader | Get virtual content type by ID |
| `[[modules/plugins-frontend/pluginLoader\|isVirtualContentType]]` | function | PluginLoader | Check if an ID is a virtual type |
| `[[modules/plugins-frontend/pluginLoader\|cleanup]]` | function | PluginLoader | Unload all plugins and reset |
| `[[modules/plugins-frontend/contentRendererRegistry\|register]]` | function | ContentRendererRegistry | Register a content renderer |
| `[[modules/plugins-frontend/contentRendererRegistry\|unregister]]` | function | ContentRendererRegistry | Unregister a content renderer by type ID |
| `[[modules/plugins-frontend/contentRendererRegistry\|unregisterPlugin]]` | function | ContentRendererRegistry | Unregister all renderers for a plugin |
| `[[modules/plugins-frontend/contentRendererRegistry\|hasCustomRenderer]]` | function | ContentRendererRegistry | Check if a custom renderer exists |
| `[[modules/plugins-frontend/contentRendererRegistry\|getRenderer]]` | function | ContentRendererRegistry | Get renderer config by type ID |
| `[[modules/plugins-frontend/contentRendererRegistry\|getAllRenderers]]` | function | ContentRendererRegistry | Get all registered renderers |
| `[[modules/plugins-frontend/contentRendererRegistry\|getRenderersForPlugin]]` | function | ContentRendererRegistry | Get renderers for a specific plugin |
| `[[modules/plugins-frontend/contentRendererRegistry\|loadComponent]]` | function | ContentRendererRegistry | Lazy-load and cache renderer component |
| `[[modules/plugins-frontend/contentRendererRegistry\|executeOnLoad]]` | function | ContentRendererRegistry | Execute onLoad hook for content type |
| `[[modules/plugins-frontend/contentRendererRegistry\|executeOnSave]]` | function | ContentRendererRegistry | Execute onSave hook for content type |
| `[[modules/plugins-frontend/contentRendererRegistry\|clearRegistry]]` | function | ContentRendererRegistry | Clear all registrations (testing) |
| `[[modules/plugins-frontend/contentRendererRegistry\|size]]` | function | ContentRendererRegistry | Count of registered renderers |
| `[[modules/plugins-frontend/contentRendererRegistry\|isPluginContentType]]` | function | ContentRendererRegistry | Check if ID starts with `plugin:` |
| `[[modules/plugins-frontend/contentRendererRegistry\|parsePluginContentTypeId]]` | function | ContentRendererRegistry | Parse `plugin:{id}:{type}` into parts |
| `[[modules/plugins-frontend/contentRendererRegistry\|buildPluginContentTypeId]]` | function | ContentRendererRegistry | Build full ID from parts |
| `[[modules/plugins-frontend/manifestSchema\|validateManifest]]` | function | manifest-schema | Validate manifest against schema |
| `[[modules/plugins-frontend/manifestSchema\|pluginManifestSchema]]` | const | manifest-schema | JSON Schema definition object |
| `[[modules/plugins-frontend/security\|checkPermission]]` | function | security | Check a single permission |
| `[[modules/plugins-frontend/security\|summarizePermissions]]` | function | security | Generate human-readable permission summary |
| `[[modules/plugins-frontend/security\|validatePluginSecurity]]` | function | security | Validate manifest security posture |
| `[[modules/plugins-frontend/security\|logPermissionRequest]]` | function | security | Audit-log a permission check |
| `[[modules/plugins-frontend/security\|getPermissionLog]]` | function | security | Retrieve audit log entries |
| `[[modules/plugins-frontend/security\|clearPermissionLog]]` | function | security | Clear the audit log |
| `[[modules/plugins-frontend/sandbox\|createSandboxedAPI]]` | function | sandbox | Create complete sandboxed API for a plugin |
| `[[modules/plugins-frontend/sandbox\|createSandboxedFetch]]` | function | sandbox | Create permission-checked fetch wrapper |
| `[[modules/plugins-frontend/sandbox\|createSandboxedStorage]]` | function | sandbox | Create namespaced storage wrapper |
| `[[modules/plugins-frontend/sandbox\|createSandboxedClipboard]]` | function | sandbox | Create permission-checked clipboard wrapper |
| `[[modules/plugins-frontend/sandbox\|PluginSandboxError]]` | class | sandbox | Base sandbox error class |
| `[[modules/plugins-frontend/sandbox\|PluginPermissionError]]` | class | sandbox | Permission denied error |
| `[[modules/plugins-frontend/sandbox\|PluginTimeoutError]]` | class | sandbox | Request timeout error |
| `[[modules/plugins-frontend/sandbox\|PluginStorageError]]` | class | sandbox | Storage operation error |
| `[[modules/plugins-frontend/sandbox\|PluginClipboardError]]` | class | sandbox | Clipboard operation error |

## Internal Structure

**Plugin Loader** — Discovery, loading, lifecycle, and state management:
- `[[modules/plugins-frontend/pluginLoader|PluginLoader.ts]]` — `pluginStore`, `discoverPlugins`, `initializePlugins`, `loadPlugin`, `loadFrontendModule`, `registerContentTypes`, `enablePlugin`, `disablePlugin`, virtual content type registry, `createPluginContext`, `createPluginAPI`, `cleanup`

**Content Renderer Registry** — Central registry for plugin content renderers:
- `[[modules/plugins-frontend/contentRendererRegistry|ContentRendererRegistry.ts]]` — Registration, lookup, lazy loading, lifecycle hooks, content type ID utilities

**Manifest Schema** — JSON Schema and validation logic:
- `[[modules/plugins-frontend/manifestSchema|manifest-schema.ts]]` — `pluginManifestSchema` definition, `validateManifest()` lightweight validator

**Security** — Permission checking and audit logging:
- `[[modules/plugins-frontend/security|security.ts]]` — `checkPermission`, `checkFileSystemPermission`, `checkNetworkPermission`, `summarizePermissions`, `validatePluginSecurity`, permission request audit log

**Sandbox** — Permission-gated API wrappers:
- `[[modules/plugins-frontend/sandbox|sandbox.ts]]` — `createSandboxedFetch`, `createSandboxedStorage`, `createSandboxedClipboard`, `createSandboxedAPI`, error classes

**Type Definitions** — TypeScript interfaces for the entire plugin system:
- `[[modules/plugins-frontend/pluginTypes|types.ts]]` — Manifest types, runtime types, lifecycle types, renderer types, context menu types, virtual content type, store types, settings types

**Barrel Export** — Clean public API:
- `[[modules/plugins-frontend/pluginIndex|index.ts]]` — Re-exports with aliased names for ergonomic consumption

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `svelte/store` | external | `writable`, `get` for reactive plugin state |
| `svelte` | external | `Component` type for renderer registration |
| `$lib/types/content` | internal | `Content` type for lifecycle hooks |
| `$lib/services/loggerService` | internal | `createLogger` for structured logging |
| `$lib/stores/notifications` | internal | `notificationStore` for user-facing messages |

## Integration Points

- **Initialization** — `initializePlugins()` is called during application startup after the backend is ready
- **Content rendering** — The editor page checks `hasCustomRenderer(contentTypeId)` to decide between TextEditor and plugin renderer
- **Content type system** — Virtual content types are merged with native types in the content tree via `getVirtualContentTypes()`
- **Plugin settings** — `PluginSettings.svelte` component uses `pluginStore` and `summarizePermissions` for the settings UI

## Logging

All modules use `createLogger` from the logging frontend service:
- `plugin-loader` — Discovery, loading, enable/disable, lifecycle hooks
- `content-renderer-registry` — Registration, component loading, hook execution
- `plugin-security` — Permission checks, security validation, audit logging
- `plugin-sandbox` — Sandboxed API creation, permission-gated operations

## Related

- [[Palimpsest-Writing-Software/features/plugins/overview\|Plugin System]]
- [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend Module]]
- [[Palimpsest-Writing-Software/api/plugins/overview\|Plugin API Service]]
