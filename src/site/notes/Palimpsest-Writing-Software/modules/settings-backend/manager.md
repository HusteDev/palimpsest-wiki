---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/settings-backend/manager/","title":"manager.go","tags":["file","settings-backend","go"],"updated":"2026-03-05T08:59:10.408-07:00"}
---


# `manager.go`

**Module:** `[[modules/settings-backend/overview|Settings Backend]]`
**Path:** `backend/internal/settings/manager.go`

> [!NOTE] Core settings storage engine providing JSON file-based persistence for system settings and user preferences, with 12-Factor environment variable overrides, input validation with safe fallbacks, field migration for backwards compatibility, thread-safe concurrent access, and DTO mapping for API isolation.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Manager` | struct | Thread-safe settings manager backed by JSON files |
| `LoggingConfig` | struct | Logging configuration with per-component level overrides |
| `ComponentConfig` | struct | Per-component logging level override |
| `New(configRoot)` | function | Constructor -- resolve paths, load files, apply env overrides, validate |

## Internal Types

These types are unexported but form the core data model persisted to disk.

### `systemSettings`

Data source: `ConfigDir/system/settings.json`

All fields are overridable by environment variables (see [Environment Variable Overrides](#environment-variable-overrides)).

| Field | Type | JSON Key | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `AppEnv` | `string` | `app_env` | `"development"` | Environment: `"development"` or `"production"` |
| `AppMode` | `string` | `app_mode` | `"local"` | Architecture: `"local"` (SQLite) or `"online"` (PostgreSQL, Release 2.0 stub) |
| `Port` | `int` | `port` | `8080` | HTTP server port (1--65535) |
| `DataDirectory` | `string` | `data_directory` | OS Documents/Palimpsest | Root for user data (projects, themes, plugins) |
| `DatabaseType` | `string` | `database_type` | `"sqlite"` | Database engine: `"sqlite"` or `"postgres"` |
| `EncryptionSalt` | `string` | `encryption_salt` | `""` | Salt for local project password encryption |
| `DevTools` | `bool` | `dev_tools` | `false` | Enable F12 console, debug endpoints, debug messages |
| `ExperimentalFeatures` | `bool` | `experimental_features` | `false` | Enable features still in development |
| `Logging` | `LoggingConfig` | `logging` | See LoggingConfig | Full logging configuration for all app layers |

### `userPreferences`

Data source: `ConfigDir/system/preferences.json`

| Field | Type | JSON Key | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `Theme` | `string` | `theme` | `"light"` | Active theme (`"light"`, `"dark"`, `"high-contrast"`, `"auto"`) |
| `Font` | `string` | `font` | `"default"` | Font mode (`"default"` or `"dyslexic"`) |
| `Locale` | `string` | `locale` | `"en"` | User locale |
| `PanelLayout` | `panelLayout` | `panel_layout` | See defaults | Layout config for left, right, bottom, and floating panels |
| `Editor` | `editorSettings` | `editor` | See defaults | Editor behavior settings |
| `WordCount` | `wordCountSettings` | `word_count` | See defaults | Word count display preferences |
| `DisableAutoNumbering` | `bool` | `disable_auto_numbering` | `false` | Disable automatic content numbering |
| `LastProjectSlug` | `string` | `last_project_slug` | `""` | Slug of the last opened project |
| `UIState` | `uiState` | `ui_state` | `{}` | Dynamic map of UI state (expanded tree IDs, collapsed panels, etc.) |
| `CreatedAt` | `time.Time` | `created_at` | now | Timestamp of initial creation |
| `UpdatedAt` | `time.Time` | `updated_at` | now | Timestamp of last modification |

### `panelLayout`

| Field | Type | Description |
| ----- | ---- | ----------- |
| `Left` | `panelConfig` | Left panel configuration |
| `Right` | `panelConfig` | Right panel configuration |
| `Bottom` | `panelConfig` | Bottom panel configuration |
| `Floating` | `[]floatingPanelInstance` | Array of floating panel instances |

### `panelConfig`

| Field | Type | Default (Left) | Description |
| ----- | ---- | --------------- | ----------- |
| `Width` | `float64` | `300` | Panel width in pixels |
| `Height` | `float64` | `0` | Panel height in pixels |
| `Visible` | `bool` | `true` | Whether the panel is visible |
| `Enabled` | `bool` | `true` | Whether the panel is enabled |
| `Components` | `[]string` | `["content-navigation"]` | Panel component IDs |
| `Addons` | `[]string` | `nil` | Legacy field -- migrated to Components on load |

### `floatingPanelInstance`

| Field | Type | Description |
| ----- | ---- | ----------- |
| `PanelID` | `string` | Unique panel identifier |
| `ComponentID` | `string` | Component displayed in the panel |
| `X` | `float64` | Horizontal position in pixels |
| `Y` | `float64` | Vertical position in pixels |
| `Width` | `float64` | Panel width in pixels |
| `Height` | `float64` | Panel height in pixels |
| `ZIndex` | `int` | Stacking order |
| `Minimized` | `bool` | Whether the panel is minimized |

### `editorSettings`

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `Spellcheck` | `bool` | `true` | Enable spellcheck |
| `GrammarCheck` | `bool` | `true` | Enable grammar check |
| `AutoSave` | `bool` | `true` | Enable timed auto-save |
| `AutoSaveInterval` | `int` | `5` | Minutes between auto-saves (1--60) |
| `WordWrap` | `bool` | `true` | Enable word wrapping |
| `SaveOnNavigation` | `string` | `"prompt"` | Behavior on navigation: `"prompt"`, `"enable"`, `"disable"` |

### `wordCountSettings`

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `ShowInTree` | `bool` | `true` | Show word counts in content tree |
| `HierarchyVisibility` | `map[string]bool` | `{}` | Per-hierarchy visibility overrides |
| `ShowWordCount` | `bool` | `true` | Show word count in status bar |
| `ShowCharCount` | `bool` | `true` | Show character count in status bar |
| `ShowPageEstimate` | `bool` | `false` | Show estimated page count |
| `CharCountMode` | `string` | `"with-spaces"` | `"with-spaces"` or `"without-spaces"` |
| `NumberFormat` | `string` | `"locale"` | `"locale"`, `"plain"`, or `"compact"` |
| `WordsPerPage` | `int` | `250` | Words per page for estimation (50--1000) |

### `uiState`

Type alias for `map[string]any`. Supports dynamic keys from the frontend. Known keys:

- `expandedTreeIds` -- `[]string` of expanded content tree node IDs
- `collapsedPanels` -- `map[string]any` with keys `left`, `right`, `bottom` (bool values)
- Dynamic keys: `activeTab_left`, `expandedSettingsSections`, etc.

## LoggingConfig

Exported struct controlling log output for all three application layers (backend, frontend, tauri).

| Field | Type | JSON Key | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `Enabled` | `bool` | `enabled` | `false` | Master kill switch for all log output |
| `DefaultLevel` | `string` | `default_level` | `"info"` | Base log level: `"debug"`, `"info"`, `"warn"`, `"error"` |
| `Format` | `string` | `format` | `"text"` | Output format: `"text"` (human-readable) or `"json"` (structured) |
| `Output` | `string` | `output` | `"console"` | Destination: `"console"`, `"file"`, or `"both"` |
| `Destination` | `string` | `destination` | `""` | Log file directory. Empty = `ConfigDir/logs/` |
| `Overwrite` | `bool` | `overwrite` | `false` | Truncate log files on startup vs timestamped files |
| `AddSource` | `bool` | `add_source` | `false` | Include Go source file:line in backend logs |
| `RetentionDays` | `int` | `retention_days` | `30` | Auto-delete logs older than N days. 0 = keep forever |
| `MaxFileSizeMB` | `int` | `max_file_size_mb` | `50` | Max file size before rotation. 0 = no limit |
| `Components` | `map[string]ComponentConfig` | `components` | `{backend, frontend, tauri}` | Per-component level overrides |

### `ComponentConfig`

| Field | Type | Description |
| ----- | ---- | ----------- |
| `Level` | `string` | Component-specific log level override. Empty = use `DefaultLevel` |

### `ResolveLevel(component)`

Returns the effective log level for a component. If the component has a non-empty level override in `Components`, that is returned. Otherwise, `DefaultLevel` is returned.

## Validation Maps

Package-level maps that define the set of valid values for enumerated fields. Used by `validateLoggingConfig()` and `validateSystemSettings()`.

| Map | Valid Values | Used By |
| --- | ------------ | ------- |
| `validLogLevels` | `debug`, `info`, `warn`, `error` | Logging default level and component overrides |
| `validOutputModes` | `console`, `file`, `both` | Logging output mode |
| `validFormats` | `text`, `json` | Logging format |
| `validAppEnvs` | `development`, `production` | System AppEnv |
| `validAppModes` | `local`, `online` | System AppMode |
| `validDatabaseTypes` | `sqlite`, `postgres` | System DatabaseType |

## Validation Functions

### `validateLoggingConfig(lc *LoggingConfig)`

Normalizes invalid or missing values in a `LoggingConfig` in place. Invalid levels fall back to `"info"`, invalid formats to `"text"`, invalid output modes to `"console"`. Negative retention/file-size values are clamped to 0. Ensures the `Components` map exists with all required keys (`backend`, `frontend`, `tauri`). Invalid component-level overrides are cleared to empty string (inherit `DefaultLevel`).

### `validateSystemSettings(s *systemSettings)`

Normalizes invalid or missing values in `systemSettings` in place. Invalid `AppEnv` falls back to `"development"`, invalid `AppMode` to `"local"`, out-of-range `Port` to `8080`, invalid `DatabaseType` to `"sqlite"`. Bool fields (`DevTools`, `ExperimentalFeatures`) need no validation. `EncryptionSalt` accepts any value (empty = no encryption).

## Environment Variable Overrides

Following 12-Factor methodology, environment variables always take precedence over `settings.json` values. Applied in memory only -- the file on disk is not modified. Called by `New()` after `loadSystemSettings()` and before validation.

| Env Var | Maps To | Notes |
| ------- | ------- | ----- |
| `APP_ENV` | `app_env` | Lowercased |
| `APP_MODE` | `app_mode` | Lowercased |
| `PORT` | `port` | Parsed as integer |
| `DATABASE_TYPE` | `database_type` | Lowercased |
| `ENCRYPTION_KEY_SALT` | `encryption_salt` | Raw string |
| `DEV_TOOLS` | `dev_tools` | Parsed as bool |
| `EXPERIMENTAL_FEATURES` | `experimental_features` | Parsed as bool |
| `LOG_ENABLED` | `logging.enabled` | Parsed as bool |
| `LOG_LEVEL` | `logging.default_level` | Lowercased |
| `LOG_FORMAT` | `logging.format` | Lowercased |
| `LOG_OUTPUT` | `logging.output` | Lowercased |
| `LOG_DESTINATION` | `logging.destination` | Raw string |
| `LOG_OVERWRITE` | `logging.overwrite` | Parsed as bool |
| `LOG_SOURCE` | `logging.add_source` | Parsed as bool |
| `LOG_RETENTION_DAYS` | `logging.retention_days` | Parsed as integer |
| `LOG_MAX_FILE_SIZE_MB` | `logging.max_file_size_mb` | Parsed as integer |

### Helper Functions

- **`envBool(value, fallback)`** -- Parses a string as boolean. Accepts `"true"`, `"1"`, `"yes"`, `"on"` as true; `"false"`, `"0"`, `"no"`, `"off"` as false. Unrecognized values return the fallback.
- **`envInt(value, fallback)`** -- Parses a string as a non-negative integer. Non-numeric characters cause a return of the fallback value.

## Constructor

### `New(configRoot string) (*Manager, error)`

Creates and initializes a new `Manager`. Performs the full startup sequence:

1. **Resolve paths** via `paths.Resolve()`. If `configRoot` is non-empty, it overrides the config directory (useful for testing).
2. **Ensure directories** exist on disk via `appPaths.EnsureDirectories()`.
3. **Load system settings** from `ConfigDir/system/settings.json` (creates with defaults if missing).
4. **Apply environment variable overrides** (12-Factor: env always wins over file).
5. **Validate** both system settings and logging config (catches bad JSON and bad env values).
6. **Load user preferences** from `ConfigDir/system/preferences.json` (creates with defaults if missing).

OS-specific default config roots:

| OS | Default Config Directory |
| -- | ------------------------ |
| Windows | `%APPDATA%\Palimpsest\` |
| macOS | `~/Library/Application Support/Palimpsest/` |
| Linux | `~/.config/palimpsest/` |

Can be overridden via the `PALIMPSEST_ROOT` environment variable.

## Settings Loading

### `loadSystemSettings()`

Loads system settings from `ConfigDir/system/settings.json`. If the file does not exist, creates it with default values and persists to disk. If the file exists but has no logging section (pre-logging-config format), the logging section is initialized with defaults. Fills in default values for any fields absent from the JSON (handles older file versions).

### `loadUserPreferences()`

Loads user preferences from `ConfigDir/system/preferences.json`. If the file does not exist, creates it with default values. On load, performs the following migrations:

- **AutoSave interval**: Values over 60 are treated as legacy millisecond values and converted to minutes.
- **SaveOnNavigation**: Empty values default to `"prompt"`.
- **Panel addons to components**: Migrates `addons` field to `components` if components is empty.
- **Word count settings**: Initializes defaults if `WordsPerPage` is 0 (indicates pre-word-count format).
- **UIState**: Initializes defaults if nil.
- **HierarchyVisibility**: Initializes empty map if nil.

## Settings Persistence

### `saveSystemSettings()`

Writes `systemSettings` to `ConfigDir/system/settings.json` with indented JSON formatting (`0o644` permissions).

### `saveUserPreferences()`

Updates `UpdatedAt` timestamp, then writes `userPreferences` to `ConfigDir/system/preferences.json` with indented JSON formatting (`0o644` permissions).

## DTO Mapping

Internal types are mapped to `repository` DTOs before being returned through the public API. This maintains isolation between the storage layer and the HTTP/API layer.

| Function | Converts | To |
| -------- | -------- | -- |
| `toLoggingDTO(lc)` | `LoggingConfig` | `repository.LoggingConfigDTO` |
| `buildPanelLayoutDTO(pl)` | `panelLayout` | `repository.PanelLayoutDTO` |
| `buildEditorDTO(es)` | `editorSettings` | `repository.EditorSettingsDTO` |
| `buildWordCountDTO(wc)` | `wordCountSettings` | `repository.WordCountSettingsDTO` |
| `buildUIStateDTO(us)` | `uiState` | `repository.UIStateDTO` |
| `buildSettingsDTO()` | All internal state | `*repository.SettingsDTO` |

`buildSettingsDTO()` is the top-level assembler. It must be called while holding at least a read lock. It combines system settings, user preferences, and timestamps into a single `SettingsDTO`.

## Public API

All public methods acquire the appropriate lock (`RLock` for reads, `Lock` for writes) and delegate to internal state. These methods implement the `repository.SettingsStore` interface.

| Method | Lock | Returns | Description |
| ------ | ---- | ------- | ----------- |
| `GetSettings()` | RLock | `(*SettingsDTO, error)` | Return merged system + user settings from in-memory cache |
| `UpdateSettings(in)` | Lock | `(*SettingsDTO, error)` | Partial update -- only non-nil fields in input are applied. Validates theme, font, autosave interval, saveOnNavigation, charCountMode, numberFormat, wordsPerPage, and data directory. Persists only changed files. |
| `ResetSettings()` | Lock | `(*SettingsDTO, error)` | Reset user preferences to defaults. System settings unchanged. Preserves original `CreatedAt` timestamp. |
| `GetSystemInfo()` | RLock | `(*SystemInfoDTO, error)` | Return system info plus project count from filesystem scan of `DataDir/Projects/*.db` |
| `GetProjectsDirectory()` | RLock | `(string, error)` | Return `DataDir/Projects` absolute path |
| `GetPaths()` | None | `*paths.Paths` | Return resolved OS-compliant paths (no lock -- immutable after construction) |
| `GetLoggingConfig()` | RLock | `LoggingConfig` | Return copy of current logging config. Used for two-phase logger init in `main.go`. |
| `GetAppEnv()` | RLock | `string` | Return effective app environment |
| `GetAppMode()` | RLock | `string` | Return effective app mode |
| `GetPort()` | RLock | `int` | Return effective HTTP server port |
| `GetDevTools()` | RLock | `bool` | Return whether dev tools are enabled |
| `GetExperimentalFeatures()` | RLock | `bool` | Return whether experimental features are enabled |
| `GetDatabaseType()` | RLock | `string` | Return effective database engine type |
| `GetEncryptionSalt()` | RLock | `string` | Return encryption salt |
| `ValidateDirectory(path)` | RLock | `(string, error)` | Validate path is valid and writable. Expands `~`, cleans path, creates directory, tests write access. |

### `UpdateSettings` Validation Rules

| Field | Validation | Fallback |
| ----- | ---------- | -------- |
| `Theme` | Must be `light`, `dark`, `high-contrast`, or `auto` | Returns error |
| `Font` | Must be `default` or `dyslexic` | Returns error |
| `AutoSaveInterval` | Must be 1--60 | Returns error |
| `SaveOnNavigation` | Must be `prompt`, `enable`, or `disable` | Defaults to `"prompt"` |
| `CharCountMode` | Must be `with-spaces` or `without-spaces` | Defaults to `"with-spaces"` |
| `NumberFormat` | Must be `locale`, `plain`, or `compact` | Defaults to `"locale"` |
| `WordsPerPage` | Must be 50--1000 | Defaults to `250` |
| `DataDirectory` | Must be writable (validated via `validateDirectoryLocked`) | Returns error |
| `Logging` | Read-only -- cannot be updated via API | Ignored |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `encoding/json` | stdlib | JSON marshal/unmarshal for settings files |
| `errors` | stdlib | Error construction and `errors.Is` for file-not-found |
| `fmt` | stdlib | Error wrapping with `fmt.Errorf` |
| `os` | stdlib | File I/O, environment variable reads, directory creation |
| `path/filepath` | stdlib | OS-safe path construction and cleaning |
| `strings` | stdlib | Input normalization (TrimSpace, ToLower, HasPrefix) |
| `sync` | stdlib | `RWMutex` for thread-safe concurrent access |
| `time` | stdlib | Timestamps on user preferences |
| `paths` | `internal/paths` | OS-compliant path resolution |
| `repository` | `internal/repository` | DTO types and `SettingsStore` interface |

## Side Effects

- **File creation**: `New()` creates `settings.json` and `preferences.json` with defaults if they do not exist.
- **Directory creation**: `New()` creates all required config/data directories via `EnsureDirectories()`.
- **Environment reads**: `applyEnvOverrides()` reads 17 environment variables on startup.
- **File writes**: `saveSystemSettings()` and `saveUserPreferences()` write to disk on every mutation.
- **Directory probing**: `ValidateDirectory()` creates the target directory and writes a temporary test file.
- **Filesystem scan**: `GetSystemInfo()` scans `DataDir/Projects/` to count `.db` files.

## Notes

> [!WARNING] Logging config is read-only via the API. To change logging settings, edit `settings.json` directly and restart the application. This is intentional -- logging config is consumed during early startup before the HTTP server is available.

> [!WARNING] The `Manager` does not use structured logging itself because it operates during application startup before the logger is initialized. The `SettingsHandler` (HTTP layer) does use injected structured logging.
