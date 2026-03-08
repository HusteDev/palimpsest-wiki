---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-backend/overview/","title":"Plugins Backend Module","tags":["module","plugins-backend"],"updated":"2026-03-05T07:52:55.430-07:00"}
---


# Plugins Backend

> [!NOTE] Go package providing plugin manifest parsing, file-system discovery of installed plugins, manifest validation with tier and capability constraints, and HTTP handlers for plugin listing, detail retrieval, and static asset serving with directory traversal protection.

## Responsibilities

- Parse `palimpsest-plugin.json` manifest files from the `DataDir/Plugins/` directory
- Validate manifests: required fields, name format (`^[a-z][a-z0-9-]*$`), semver version, tier constraints, content type rules
- Discover all plugins by scanning subdirectories, including invalid plugins with error messages
- Serve plugin HTTP endpoints: list all plugins, get single plugin, serve static assets
- Protect against directory traversal in asset paths
- Define Go type system for plugin manifests, capabilities, permissions, and discovery results

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `[[modules/plugins-backend/types\|PluginManifest]]` | struct | Complete plugin manifest from `palimpsest-plugin.json` |
| `[[modules/plugins-backend/types\|PluginAuthor]]` | struct | Author information (name, email, URL) |
| `[[modules/plugins-backend/types\|PluginCapabilities]]` | struct | Boolean flags for each extension point |
| `[[modules/plugins-backend/types\|PluginPermissions]]` | struct | Permission declarations (fileSystem, network, localStorage, clipboard) |
| `[[modules/plugins-backend/types\|PluginEntryPoints]]` | struct | Frontend and optional backend code paths |
| `[[modules/plugins-backend/types\|PluginContentType]]` | struct | Custom content type definition |
| `[[modules/plugins-backend/types\|PluginPanel]]` | struct | Panel component definition |
| `[[modules/plugins-backend/types\|PluginContextMenuItem]]` | struct | Context menu item definition |
| `[[modules/plugins-backend/types\|PluginJobHandler]]` | struct | Backend job handler definition (Pro only) |
| `[[modules/plugins-backend/types\|PluginRestEndpoint]]` | struct | REST endpoint definition (Pro only) |
| `[[modules/plugins-backend/types\|PluginSource]]` | type | String enum: `local`, `npm`, `marketplace` |
| `[[modules/plugins-backend/types\|PluginStatus]]` | type | String enum: `loading`, `loaded`, `enabled`, `disabled`, `error` |
| `[[modules/plugins-backend/types\|DiscoveredPlugin]]` | struct | Discovery result with manifest, path, source, and optional error |
| `[[modules/plugins-backend/types\|PluginListResponse]]` | struct | Response wrapper for `GET /api/plugins` |
| `[[modules/plugins-backend/discovery\|ManifestFileName]]` | const | `"palimpsest-plugin.json"` |
| `[[modules/plugins-backend/discovery\|Discovery]]` | struct | Plugin discovery service with directory scanning |
| `[[modules/plugins-backend/discovery\|NewDiscovery]]` | function | Create discovery service for a plugins directory |
| `[[modules/plugins-backend/discovery\|Discovery.DiscoverAll]]` | method | Scan and return all discovered plugins |
| `[[modules/plugins-backend/discovery\|Discovery.GetPluginPath]]` | method | Get absolute path to a plugin directory |
| `[[modules/plugins-backend/discovery\|Discovery.GetPluginAssetPath]]` | method | Get absolute path to a plugin asset file |
| `[[modules/plugins-backend/discovery\|Discovery.PluginExists]]` | method | Check if a plugin directory exists |
| `[[modules/plugins-backend/pluginsHandler\|PluginHandler]]` | struct | HTTP handler for all plugin endpoints |
| `[[modules/plugins-backend/pluginsHandler\|NewPluginHandler]]` | function | Create handler with discovery service |

## Internal Structure

**Type Definitions** — Go type system mirroring the TypeScript manifest interfaces:
- `[[modules/plugins-backend/types|types.go]]` — All plugin-related structs, type aliases, and constants

**Discovery Service** — File-system scanning and manifest validation:
- `[[modules/plugins-backend/discovery|discovery.go]]` — `Discovery` struct, `DiscoverAll`, `discoverPlugin`, `validateManifest`, `validateContentType`, path helpers

**HTTP Handlers** — Plugin API endpoints:
- `[[modules/plugins-backend/pluginsHandler|plugins.go]]` — `PluginHandler` struct, `Plugins` router, `ListPlugins`, `GetPlugin`, `ServePluginAsset`, `serveFile`

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `encoding/json` | external (stdlib) | Parse manifest JSON |
| `os` | external (stdlib) | File system operations, directory scanning |
| `path/filepath` | external (stdlib) | Path construction and joining |
| `regexp` | external (stdlib) | Name and version format validation |
| `net/http` | external (stdlib) | HTTP handler interfaces |
| `mime` | external (stdlib) | MIME type detection for asset serving |
| `io` | external (stdlib) | Stream file content to response |
| `logging` | internal | Structured logging via injected logger pattern |
| `response` | internal | HTTP response helpers (`OK`, `NotFound`, `BadRequest`, etc.) |

## Integration Points

- **Initialization** — `NewDiscovery(pluginsDir)` is called during backend startup with the resolved `DataDir/Plugins/` path
- **HTTP routing** — `PluginHandler.Plugins()` is mounted at `/api/plugins` and handles all sub-routes
- **Frontend consumption** — The Svelte `PluginLoader` fetches `GET /api/plugins` to discover available plugins
- **Asset serving** — Plugin frontend bundles are served from `/api/plugins/{name}/assets/{path}` for dynamic import

## Logging

Uses the injected logger pattern per `CLAUDE.md`:
- `Discovery` struct: `logging.GetGlobal().WithComponent("plugin-discovery")`
- `PluginHandler` struct: `logging.GetGlobal().WithComponent("handler-plugins")`

Message prefixes: `[DiscoverAll]`, `[ListPlugins]`, `[GetPlugin]`, `[ServePluginAsset]`

## Security

- **Directory traversal protection** — `ServePluginAsset` rejects any asset path containing `..` with a 400 Bad Request and logs a warning
- **Manifest validation** — Enforces name format, version format, tier constraints, and content type rules before the frontend ever sees plugin data
- **Plugin existence check** — All asset and detail endpoints verify the plugin directory exists before serving

## Related

- [[Palimpsest-Writing-Software/features/plugins/overview\|Plugin System]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/overview\|Plugins Frontend Module]]
- [[Palimpsest-Writing-Software/api/plugins/overview\|Plugin API Service]]
