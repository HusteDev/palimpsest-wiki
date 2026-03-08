---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/settings-frontend/system-settings/","title":"systemSettings.ts","tags":["file","settings-frontend","typescript","svelte","store"],"updated":"2026-03-05T08:58:08.404-07:00"}
---


# `systemSettings.ts`

**Module:** [[Palimpsest-Writing-Software/modules/settings-frontend/overview\|Settings Frontend]]
**Path:** `src/lib/stores/systemSettings.ts`

> [!NOTE] Read-only Svelte store for system-level infrastructure settings loaded from the backend API. Provides reactive access to application environment, logging configuration, and developer tool flags. Frontend cannot mutate these values -- they must be edited directly in `settings.json` and require an app restart.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ComponentLogging` | interface | Per-component logging level override |
| `LoggingConfig` | interface | Full logging configuration for all application layers |
| `SystemSettings` | interface | Top-level system settings loaded from `settings.json` |
| `devToolsEnabled` | derived store | Reactive boolean derived from `SystemSettings.devTools` |
| `systemSettings` | object | Primary export: subscribable store with load method and convenience getters |

## `ComponentLogging` Interface

Per-component logging override. Used as values in `LoggingConfig.components`.

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `level` | `string` | yes | Component-specific log level. Empty string means "use the `defaultLevel`" |

## `LoggingConfig` Interface

Full logging configuration from `settings.json`. Controls log output for all three application layers (backend, frontend, Tauri). Read-only from the frontend.

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `enabled` | `boolean` | yes | `false` | Master kill switch -- `false` suppresses all log output |
| `defaultLevel` | `string` | yes | `'info'` | Base log level for all components: `"debug"`, `"info"`, `"warn"`, `"error"` |
| `format` | `string` | yes | `'text'` | Output format: `"text"` (human-readable) or `"json"` (structured) |
| `output` | `string` | yes | `'console'` | Where logs go: `"console"`, `"file"`, or `"both"` |
| `destination` | `string` | yes | `''` | Log directory path. Empty means default (`ConfigDir/logs/`) |
| `overwrite` | `boolean` | yes | `false` | If `true`, truncate on startup. If `false`, create timestamped files |
| `addSource` | `boolean` | yes | `false` | If `true`, include Go source `file:line` in backend log entries |
| `retentionDays` | `number` | yes | `30` | Auto-delete log files older than N days. `0` = keep forever |
| `maxFileSizeMB` | `number` | yes | `50` | Max file size in MB before rotation. `0` = no limit |
| `components` | `Record<string, ComponentLogging>` | yes | see below | Per-component level overrides keyed by component name |

**Default `components`:**

| Key | Level |
| --- | ----- |
| `backend` | `''` (use defaultLevel) |
| `frontend` | `''` (use defaultLevel) |
| `tauri` | `''` (use defaultLevel) |

## `SystemSettings` Interface

System-wide settings loaded from the backend API. Corresponds to the system section of `settings.json`.

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `appEnv` | `string` | yes | `'development'` | Application environment: `"development"` or `"production"` |
| `appMode` | `string` | yes | `'local'` | Application mode: `"local"` or `"online"` |
| `port` | `number` | yes | `8080` | HTTP server port for the backend API |
| `dataDirectory` | `string` | yes | `''` | Root directory for projects, themes, and plugins |
| `databaseType` | `string` | yes | `'sqlite'` | Database engine: `"sqlite"` or `"postgres"` |
| `encryptionSalt` | `string` | yes | `''` | Encryption salt for local project passwords |
| `devTools` | `boolean` | yes | `false` | Enable developer tools: F12 console, debug messages, JSON viewer |
| `experimentalFeatures` | `boolean` | yes | `false` | Enable experimental features (not recommended for normal use) |
| `logging` | `LoggingConfig` | yes | `defaultLogging` | Full logging configuration for all application layers |

## Constants

### `defaultLogging`

Default `LoggingConfig` used when the backend API does not return a logging configuration or on load failure. Logging is disabled by default with `info` level, text format, console output, 30-day retention, and 50 MB max file size.

### `defaultSettings`

Default `SystemSettings` used as the initial store value and as the fallback on load failure. Sets a development environment in local mode on port 8080 with SQLite, devTools off, and experimental features off.

## Internal Stores

| Store | Type | Initial Value | Description |
| ----- | ---- | ------------- | ----------- |
| `settings` | `writable<SystemSettings>` | `defaultSettings` | Primary internal store holding current system settings |
| `isLoaded` | `writable<boolean>` | `false` | Tracks whether `loadSettings()` has completed (success or failure) |

## `loadSettings` Behavior

```
loadSettings() -> Promise<void>
```

Fetches system settings from the backend API and populates the internal store.

**Data source:** Backend API (`GET /api/settings` -> `settings.json`)

**Steps:**

1. **Fetch** -- Calls `apiClient.get<{ data: any }>('/api/settings')` to retrieve the settings JSON.
2. **Map** -- Maps each field from the API response to the `SystemSettings` interface. Uses `||` fallback for string fields and `??` fallback for boolean/object fields to ensure defaults are applied for missing or falsy values.
3. **Set loaded** -- Sets `isLoaded` to `true` so consumers know the store is ready.
4. **Error handling** -- On failure, logs a raw `console.warn` (logger may not be initialized yet) and keeps the default settings. Still sets `isLoaded` to `true` so the app can function with defaults.

> [!WARNING] Uses `console.warn` directly instead of the logger service because `loadSettings()` runs before `initLogger()` during app startup. The logger service itself depends on the `LoggingConfig` from this store.

## `devToolsEnabled` Derived Store

```ts
export const devToolsEnabled = derived(settings, ($settings) => $settings.devTools);
```

A reactive boolean store derived from the internal `settings` store. Emits `true` when developer tools are enabled. Used by components that conditionally render debug UI or developer-only features.

## `systemSettings` Exported Object

The primary public API. Combines store subscription, load action, and convenience getters into a single export.

| Member | Kind | Returns | Description |
| ------ | ---- | ------- | ----------- |
| `subscribe` | store method | `Unsubscriber` | Svelte store contract -- subscribe to `SystemSettings` changes |
| `isLoaded` | sub-store | `{ subscribe }` | Subscribe to load completion status |
| `load` | method | `Promise<void>` | Alias for `loadSettings()` |
| `devTools` | getter | `boolean` | Synchronous snapshot of current `devTools` value |
| `logging` | getter | `LoggingConfig` | Synchronous snapshot of current logging configuration |

**Getter pattern:** Both `devTools` and `logging` use an immediate subscribe-unsubscribe pattern to extract the current value synchronously. The subscribe callback captures the value, and the returned unsubscribe function is called immediately (the `()` at the end of `settings.subscribe(...)()`) to prevent memory leaks.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `writable`, `derived` | `svelte/store` | Reactive store primitives |
| `apiClient` | `$lib/api/client` | HTTP client for `GET /api/settings` |
| `createLogger` | `$lib/services/loggerService` | Logger instance (component name: `'settings'`) |

## Side Effects

- Creates a module-level logger via `createLogger('settings')`, though it is not used in the current implementation (raw `console.warn` is used instead for the bootstrap timing reason noted above)
- The `isLoaded` store is set to `true` on both success and failure, meaning consumers cannot distinguish between "loaded with defaults" and "loaded from API"

## Notes

> [!WARNING] This store is strictly read-only in the frontend. There is no `save` or `update` method. To change system settings, edit `settings.json` in the data directory and restart the application. This is by design -- system settings control infrastructure concerns (ports, database type, encryption) that should not be modifiable through the UI.
