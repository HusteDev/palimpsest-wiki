---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-backend/discovery/","title":"discovery.go","tags":["file","plugins-backend"],"updated":"2026-03-05T07:55:46.409-07:00"}
---


# `discovery.go`

**Module:** [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend]]
**Path:** `backend/internal/plugins/discovery.go`

> [!NOTE] Plugin discovery service that scans `DataDir/Plugins/` for plugin directories containing a `palimpsest-plugin.json` manifest, parses and validates each manifest, and returns the full set of discovered plugins (including invalid ones with error details).

## Exports

### Constants

| Name | Kind | Value | Description |
| ---- | ---- | ----- | ----------- |
| `ManifestFileName` | const (`string`) | `"palimpsest-plugin.json"` | Expected filename for plugin manifest files. Every plugin directory must contain a file with this name. |

### Struct

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Discovery` | struct | Plugin discovery service. Holds `pluginsDir` (string, absolute path to the plugins directory) and `log` (*logging.Logger, component `"plugin-discovery"`). |

### Functions / Methods

| Name | Kind | Parameters | Returns | Description |
| ---- | ---- | ---------- | ------- | ----------- |
| `NewDiscovery` | constructor | `pluginsDir string` -- absolute path to the plugins directory (e.g., `DataDir/Plugins`) | `*Discovery` | Creates a new Discovery service instance with an injected logger using the `"plugin-discovery"` component name. |
| `DiscoverAll` | method on `*Discovery` | _(none)_ | `([]DiscoveredPlugin, error)` | Scans the plugins directory for subdirectories, reads and validates each manifest. Returns all discovered plugins (valid and invalid). Invalid plugins are included with their `Error` field populated rather than being silently skipped. Returns a fatal `error` only if the directory itself cannot be read. Returns an empty slice (not nil) if the plugins directory does not exist. |
| `discoverPlugin` | method on `*Discovery` (unexported) | `pluginPath string` -- absolute path to a single plugin directory | `(DiscoveredPlugin, error)` | Reads the manifest file from the given plugin directory, parses the JSON, and validates it via `validateManifest`. Returns a fully populated `DiscoveredPlugin` on success or an error describing what went wrong. |
| `validateManifest` | method on `*Discovery` (unexported) | `manifest *PluginManifest` -- pointer to the manifest to validate | `error` | Performs validation on a parsed manifest. Checks: required fields (`name`, `displayName`, `version`, `palimpsestVersion`, `description`, `tier`, `entryPoints.frontend`); name format (regex `^[a-z][a-z0-9-]*$`); version format (semver regex); tier value (`"core"` or `"pro"`); backend capabilities (`jobHandlers`, `restEndpoints`) require `tier: "pro"`; contentTypes capability enabled requires at least one content type defined. Delegates to `validateContentType` for each content type entry. |
| `validateContentType` | method on `*Discovery` (unexported) | `ct *PluginContentType` -- the content type to validate; `index int` -- array index for error messages | `error` | Validates a single content type definition. Checks: required fields (`id`, `name`, `icon`, `renderer`); ID format (regex `^[a-z][a-z0-9-]*$`); `hasContent` must be `true`; `canHaveChildren` must be `false`. |
| `GetPluginPath` | method on `*Discovery` | `pluginName string` -- the plugin identifier | `string` | Returns the absolute path to a plugin's directory by joining `pluginsDir` and `pluginName`. |
| `GetPluginAssetPath` | method on `*Discovery` | `pluginName string` -- the plugin identifier; `assetPath string` -- relative path within the plugin directory | `string` | Returns the absolute path to a specific asset file within a plugin directory. Used for serving frontend bundles and other static assets. |
| `PluginExists` | method on `*Discovery` | `pluginName string` -- the plugin identifier | `bool` | Checks if a plugin directory exists on disk and is actually a directory (not a file). Returns `false` on any `os.Stat` error. |

## Validation Rules Summary

### Manifest-Level Rules

| Rule | Detail |
| ---- | ------ |
| Required fields | `name`, `displayName`, `version`, `palimpsestVersion`, `description`, `tier`, `entryPoints.frontend` |
| Name format | Must match `^[a-z][a-z0-9-]*$` (lowercase, starts with letter, alphanumeric + hyphens) |
| Version format | Must be semantic version: `^\d+\.\d+\.\d+(-[a-zA-Z0-9.]+)?(\+[a-zA-Z0-9.]+)?$` |
| Tier values | Must be `"core"` or `"pro"` |
| Pro-tier constraint | `jobHandlers` or `restEndpoints` capabilities require `tier: "pro"` |
| ContentTypes coherence | If `capabilities.contentTypes` is `true`, at least one content type must be defined |

### Content-Type-Level Rules

| Rule | Detail |
| ---- | ------ |
| Required fields | `id`, `name`, `icon`, `renderer` |
| ID format | Must match `^[a-z][a-z0-9-]*$` |
| `hasContent` | Must be `true` for plugin content types |
| `canHaveChildren` | Must be `false` for plugin content types |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `encoding/json` | stdlib | Unmarshal manifest JSON |
| `fmt` | stdlib | Error formatting |
| `os` | stdlib | File system operations (`Stat`, `ReadDir`, `ReadFile`) |
| `path/filepath` | stdlib | Cross-platform path joining |
| `regexp` | stdlib | Name and version format validation |
| `logging` | `backend/internal/logging` | Structured logging via injected `*logging.Logger` with component `"plugin-discovery"` |

## Side Effects

- **Directory scanning:** `DiscoverAll` reads the file system at `DataDir/Plugins/`, iterating subdirectories and reading manifest files. This is read-only I/O with no writes.
- **Logging:** All methods emit structured log messages at DEBUG, INFO, WARN, and ERROR levels through the injected logger.

## Notes

> [!WARNING] `DiscoverAll` does **not** short-circuit on individual plugin errors. Invalid plugins are included in the returned slice with their `Error` field set. Callers must check the `Error` field on each `DiscoveredPlugin` to distinguish valid from invalid plugins.

> [!WARNING] The `discoverPlugin` method always sets `Source` to `PluginSourceLocal`. Support for NPM and marketplace sources is declared in [[Palimpsest-Writing-Software/modules/plugins-backend/types\|types.go]] but not yet implemented in the discovery path.

> [!WARNING] Regex compilation for name and version validation happens on every call to `validateManifest` and `validateContentType`. If discovery is called frequently, consider compiling these regexes at package or struct level.
