---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/plugin-types/","title":"types.ts","tags":["file","plugins-frontend"],"updated":"2026-03-05T07:55:41.398-07:00"}
---


# `types.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/types.ts`

> [!NOTE] Central type definitions for the Palimpsest plugin system, covering manifest schema, runtime state, lifecycle hooks, renderer configuration, and settings interfaces.

## Exports

| Name | Kind | Description |
| --- | --- | --- |
| `PluginAuthor` | interface | Author metadata: `name`, optional `email` and `url` |
| `PluginCapabilities` | interface | Boolean flags declaring which extension points a plugin uses (`contentTypes`, `panels`, `contextMenuItems`, `jobHandlers`, `restEndpoints`) |
| `PluginPermissions` | interface | Permission declarations for `fileSystem` (boolean or path array), `network` (boolean or domain array), `localStorage`, and `clipboard` |
| `PluginEntryPoints` | interface | Code entry points: required `frontend` path, optional `backend` path (Pro only) |
| `PluginContentTypeDefinition` | interface | Manifest definition for a custom content type: `id`, `name`, `icon`, `color`, `hasContent` (always `true`), `canHaveChildren` (always `false`), `canBeChildOf`, `renderer` name, optional `dataSchema` |
| `PluginPanelDefinition` | interface | Manifest definition for a sidebar/floating panel: `id`, `name`, `icon`, `defaultPosition`, `allowedPositions`, `component` name |
| `PluginContextMenuItemDefinition` | interface | Manifest definition for a context menu item: `id`, `label`, `icon`, `contentTypes` filter, `handler` name, optional `shortcut` |
| `PluginJobHandlerDefinition` | interface | Backend job handler definition (Pro only): `type`, `name`, `handler` |
| `PluginRestEndpointDefinition` | interface | REST endpoint definition (Pro only): `method`, `path`, `handler` |
| `PluginTier` | type | Union literal `'core' \| 'pro'` representing the required license tier |
| `PluginManifest` | interface | Complete manifest structure for `palimpsest-plugin.json`. Contains all required fields (`name`, `displayName`, `version`, `palimpsestVersion`, `description`, `capabilities`, `permissions`, `tier`, `entryPoints`) and optional arrays for `contentTypes`, `panels`, `contextMenuItems`, `jobHandlers`, `restEndpoints` |
| `PluginSource` | type | Union literal `'local' \| 'npm' \| 'marketplace'` indicating where a plugin was loaded from |
| `PluginStatus` | type | Union literal `'loading' \| 'loaded' \| 'enabled' \| 'disabled' \| 'error'` representing a plugin's runtime state |
| `LoadedPlugin` | interface | Runtime representation of a loaded plugin: `manifest`, `source`, `path`, `status`, optional `error`, optional `frontendModule` |
| `PluginFrontendModule` | interface | Expected shape of a plugin's frontend entry point exports. Uses an index signature keyed by renderer name. May contain an optional `lifecycle` property |
| `PluginContext` | interface | Context passed to lifecycle hooks: `pluginId`, `dataDir`, `settings` record, and `api` (`PluginAPI`) |
| `PluginAPI` | interface | API surface available to plugins with sub-objects: `content` (`get`/`update`), `notifications` (`success`/`error`/`info`/`warning`), `events` (`on`/`dispatch`), `storage` (`get`/`set`/`remove`) |
| `CreatePluginContentInput` | interface | Input for creating plugin content: `title`, `contentTypeId`, `content`, optional `customFields` |
| `PluginLifecycle` | interface | Lifecycle hooks a plugin can implement (10 hooks): `onInstall`, `onUninstall`, `onEnable`, `onDisable`, `onProjectOpen`, `onProjectClose`, `onContentCreate`, `onContentLoad`, `onContentSave`, `onContentDelete` |
| `PluginRendererProps` | interface | Props passed to renderer components: `content` (Content item), `data`, `editable` boolean, `onupdate` callback |
| `ContentRendererConfig` | interface | Configuration for a registered renderer: `contentTypeId`, `pluginId`, `component` (lazy-load factory), optional `onLoad`/`onSave` hooks |
| `ContextMenuContext` | interface | Context passed to context menu handlers: `content`, `projectSlug`, `pluginContext` |
| `PluginContextMenuHandler` | type | Function signature `(context: ContextMenuContext) => Promise<void>` |
| `VirtualContentType` | interface | Frontend-only content type registered by a plugin. Fixed fields: `hierarchyName: 'plugin'`, `level: 999`, `hasContent: true`, `isSystem: false`, `isVirtual: true`. Includes `metadata` with icon, color, labels, and `canBeChildOf` constraint |
| `PluginStoreState` | interface | Svelte store state: `plugins` Map, `initialized` boolean, `loading` boolean, optional `lastError` |
| `PluginUserSettings` | interface | Per-plugin user settings: `enabled`, `permissionsApproved`, `settings` record |
| `PluginSettings` | interface | All plugin settings indexed by plugin name, each value a `PluginUserSettings` |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| `Component` | `svelte` | Type reference for Svelte component in `PluginFrontendModule` and `ContentRendererConfig` |
| `Content` | `[[modules/content-frontend/overview\|content]]` | Type reference for content items used throughout lifecycle hooks and renderer props |

## Side Effects

None. This file is purely type definitions and produces no runtime code after TypeScript compilation.

## Notes

> [!WARNING] `PluginContentTypeDefinition` enforces `hasContent: true` and `canHaveChildren: false` at the type level. Plugin content types are always leaf nodes in the content hierarchy.

> [!WARNING] `VirtualContentType` uses a fixed `level` of `999` and `hierarchyName` of `'plugin'`. These values ensure plugin types sort below all built-in content types and are identifiable as virtual.

> [!WARNING] The `PluginAPI.events` sub-object (`on`/`dispatch`) is defined in the type but currently implemented as stubs in `PluginLoader.ts`. Full event bus integration is planned for a future release.
