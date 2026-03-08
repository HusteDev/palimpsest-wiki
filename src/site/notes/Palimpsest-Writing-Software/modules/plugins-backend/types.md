---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-backend/types/","title":"types.go","tags":["file","plugins-backend"],"updated":"2026-03-05T07:55:13.408-07:00"}
---


# `types.go`

**Module:** [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend]]
**Path:** `backend/internal/plugins/types.go`

> [!NOTE] Go type definitions for the Palimpsest plugin system. Mirrors the TypeScript plugin manifest types for backend validation and discovery. All structs use JSON tags for serialization to and from `palimpsest-plugin.json` manifest files.

## Exports

### Structs

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `PluginManifest` | struct | Top-level representation of a plugin's `palimpsest-plugin.json` manifest file. Contains identity fields (name, version, tier), capability declarations, entry points, and optional extension definitions (content types, panels, context menus, job handlers, REST endpoints). |
| `PluginAuthor` | struct | Optional author metadata: `Name` (string), `Email` (string, omitempty), `URL` (string, omitempty). |
| `PluginCapabilities` | struct | Boolean flags declaring which extension points the plugin uses: `ContentTypes`, `Panels`, `ContextMenuItems`, `JobHandlers`, `RestEndpoints`. |
| `PluginPermissions` | struct | Permission declarations. `FileSystem` and `Network` are `interface{}` to handle `false`, `true`, or string-array values during JSON unmarshaling. `LocalStorage` and `Clipboard` are simple booleans. |
| `PluginEntryPoints` | struct | Paths to plugin code. `Frontend` (required) is the path to the frontend JavaScript bundle. `Backend` (optional, omitempty) is the path to a backend Go plugin binary. |
| `PluginContentType` | struct | Defines a custom content type registered by a plugin. Includes `ID`, `Name`, `Icon`, `Color`, `HasContent` (must be true), `CanHaveChildren` (must be false), `CanBeChildOf` (interface{}: `"*"` or string array), `Renderer` (exported component name), and optional `DataSchema` (JSON Schema map). |
| `PluginPanel` | struct | Defines a custom panel component: `ID`, `Name`, `Icon`, `DefaultPosition`, `AllowedPositions` (string slice), `Component` (exported component name). |
| `PluginContextMenuItem` | struct | Defines a context menu addition: `ID`, `Label`, `Icon`, `ContentTypes` (interface{}: `"*"` or string array), `Handler` (function name), optional `Shortcut`. |
| `PluginJobHandler` | struct | Defines a backend job handler (Pro only): `Type`, `Name`, `Handler`. |
| `PluginRestEndpoint` | struct | Defines a custom REST endpoint (Pro only): `Method`, `Path`, `Handler`. |
| `DiscoveredPlugin` | struct | Represents a plugin found during discovery. Contains `Name`, `Path` (absolute directory path), `Source` (PluginSource), `Manifest` (PluginManifest), and optional `Error` string for invalid plugins. |
| `PluginListResponse` | struct | API response wrapper for `GET /api/plugins`. Contains a single `Plugins` field (slice of `DiscoveredPlugin`). |

### Type Aliases and Constants

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `PluginSource` | type alias (`string`) | Indicates where a plugin was loaded from. |
| `PluginSourceLocal` | const (`PluginSource`) | Value `"local"` -- plugin loaded from `DataDir/Plugins/`. |
| `PluginSourceNPM` | const (`PluginSource`) | Value `"npm"` -- plugin loaded from `node_modules` (dev mode). |
| `PluginSourceMarketplace` | const (`PluginSource`) | Value `"marketplace"` -- plugin downloaded from the marketplace. |
| `PluginStatus` | type alias (`string`) | Indicates the current runtime state of a plugin. |
| `PluginStatusLoading` | const (`PluginStatus`) | Value `"loading"`. |
| `PluginStatusLoaded` | const (`PluginStatus`) | Value `"loaded"`. |
| `PluginStatusEnabled` | const (`PluginStatus`) | Value `"enabled"`. |
| `PluginStatusDisabled` | const (`PluginStatus`) | Value `"disabled"`. |
| `PluginStatusError` | const (`PluginStatus`) | Value `"error"`. |

## PluginManifest Fields

| Field | Type | JSON Key | Required | Description |
| ----- | ---- | -------- | -------- | ----------- |
| `Name` | `string` | `name` | yes | Plugin identifier (lowercase, no spaces). |
| `DisplayName` | `string` | `displayName` | yes | Human-readable plugin name. |
| `Version` | `string` | `version` | yes | Semantic version (e.g., `"1.0.0"`). |
| `PalimpsestVersion` | `string` | `palimpsestVersion` | yes | Required Palimpsest version range. |
| `Description` | `string` | `description` | yes | Short description of what the plugin does. |
| `Author` | `*PluginAuthor` | `author` | no | Optional author information. |
| `License` | `string` | `license` | no | SPDX license identifier. |
| `Repository` | `string` | `repository` | no | Source repository URL. |
| `Capabilities` | `PluginCapabilities` | `capabilities` | yes | Declares which extension points the plugin uses. |
| `Permissions` | `PluginPermissions` | `permissions` | yes | Declares required permissions. |
| `Tier` | `string` | `tier` | yes | License tier: `"core"` or `"pro"`. |
| `EntryPoints` | `PluginEntryPoints` | `entryPoints` | yes | Paths to plugin code. |
| `ContentTypes` | `[]PluginContentType` | `contentTypes` | no | Custom content types the plugin registers. |
| `Panels` | `[]PluginPanel` | `panels` | no | Custom panel component definitions. |
| `ContextMenuItems` | `[]PluginContextMenuItem` | `contextMenuItems` | no | Context menu additions. |
| `JobHandlers` | `[]PluginJobHandler` | `jobHandlers` | no | Backend job handlers (Pro only). |
| `RestEndpoints` | `[]PluginRestEndpoint` | `restEndpoints` | no | Custom REST endpoints (Pro only). |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| _(none)_ | -- | This file is a pure type-definition file with no imports. |

## Side Effects

None. This file only defines types and constants; it performs no I/O, no registration, and no global state mutation.

## Notes

> [!WARNING] `PluginPermissions.FileSystem` and `PluginPermissions.Network` are typed as `interface{}` because the JSON manifest allows `false`, `true`, or an array of path/URL strings. Consumers must perform type assertions when reading these fields.

> [!WARNING] `PluginContentType.CanBeChildOf` is also `interface{}` for the same reason -- it can be the string `"*"` (any parent) or an array of content type ID strings. Always type-check before use.

> [!WARNING] `JobHandlers` and `RestEndpoints` are Pro-tier-only capabilities. Validation in [[Palimpsest-Writing-Software/modules/plugins-backend/discovery\|discovery.go]] enforces that these capabilities require `tier: "pro"`.
