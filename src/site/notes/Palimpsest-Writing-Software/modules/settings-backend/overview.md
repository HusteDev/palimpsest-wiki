---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/settings-backend/overview/","title":"Settings Backend Module","tags":["module","settings-backend"],"updated":"2026-03-05T08:55:42.442-07:00"}
---


# Settings Backend

> [!NOTE] Go package providing JSON file-based settings storage with two-tier architecture (system settings + user preferences), 12-Factor environment variable overrides, input validation with safe fallbacks, field migration for backwards compatibility, thread-safe concurrent access via RWMutex, and DTO mapping to a unified API response.

## Responsibilities

- Load, validate, and persist system settings from `ConfigDir/system/settings.json`
- Load, validate, migrate, and persist user preferences from `ConfigDir/system/preferences.json`
- Apply environment variable overrides for all system settings (12-Factor compliance)
- Validate all inputs (theme whitelist, font enum, autosave range 1–60, directory writability, log level/format/output enums)
- Normalize invalid/missing values to safe defaults rather than returning errors
- Migrate legacy field formats (addons → components, ms → minutes, missing sections)
- Map internal types to DTOs via `repository.SettingsStore` interface
- Provide thread-safe read/write access via `sync.RWMutex`
- Serve HTTP endpoints for settings CRUD, reset, system info, and directory validation

## Public API Surface

### Manager (settings.Manager)

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `New(configRoot)` | constructor | Create Manager, load files, apply env overrides, validate |
| `GetSettings()` | method | Return merged `SettingsDTO` (system + user) |
| `UpdateSettings(in)` | method | Partial update with validation and selective persistence |
| `ResetSettings()` | method | Reset user preferences to defaults (system unchanged) |
| `GetSystemInfo()` | method | Return system info including project count from filesystem |
| `GetProjectsDirectory()` | method | Return resolved `DataDir/Projects` path |
| `ValidateDirectory(path)` | method | Validate directory exists, is writable, expand `~` |
| `GetPaths()` | method | Return resolved OS-compliant `*paths.Paths` |
| `GetLoggingConfig()` | method | Return copy of current logging configuration |
| `GetAppEnv()` | method | Return effective app environment |
| `GetAppMode()` | method | Return effective app mode |
| `GetPort()` | method | Return effective HTTP server port |
| `GetDevTools()` | method | Return whether dev tools are enabled |
| `GetExperimentalFeatures()` | method | Return whether experimental features are enabled |
| `GetDatabaseType()` | method | Return effective database engine type |
| `GetEncryptionSalt()` | method | Return encryption salt |

### Types

| Type | Kind | Description |
| ---- | ---- | ----------- |
| `LoggingConfig` | struct | Logging configuration (enabled, level, format, output, retention, components) |
| `ComponentConfig` | struct | Per-component logging override (level) |
| `LoggingConfig.ResolveLevel(component)` | method | Resolve effective level for a component |

### HTTP Handler (handlers.SettingsHandler)

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `NewSettingsHandler(settings)` | constructor | Create handler with settings store |
| `Settings(w, r)` | method | Router: GET → GetSettings, PUT → UpdateSettings |
| `GetSettings(w, r)` | handler | `GET /api/settings` — return merged settings |
| `UpdateSettings(w, r)` | handler | `PUT /api/settings` — partial update |
| `ResetSettings(w, r)` | handler | `POST /api/settings/reset` — reset to defaults |
| `GetSystemInfo(w, r)` | handler | `GET /api/settings/system` — system information |
| `ValidateDirectory(w, r)` | handler | `POST /api/settings/validate-directory` — check path |
| `GetProjectsDirectory(w, r)` | handler | `GET /api/settings/projects-directory` — resolved path |

## Internal Structure

**Settings Manager** — Core storage and business logic:
- `[[modules/settings-backend/manager|manager.go]]` — `Manager` struct, all types (LoggingConfig, systemSettings, userPreferences, panelLayout, editorSettings, wordCountSettings, uiState), environment overrides, validation, loading, persistence, DTO mapping, public API

**HTTP Handler** — API endpoints:
- `[[modules/settings-backend/settingsHandler|settings.go]]` — `SettingsHandler` struct, HTTP routing for all settings endpoints

**Repository Interface** — DTO isolation layer:
- `repository/settings.go` — `SettingsStore` interface, `SettingsDTO`, `UpdateSettingsInput`, `SystemInfoDTO`, and all nested DTOs

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `encoding/json` | external (stdlib) | JSON serialization/deserialization |
| `os` | external (stdlib) | File I/O, environment variable reads |
| `path/filepath` | external (stdlib) | Path construction |
| `sync` | external (stdlib) | RWMutex for concurrent access |
| `time` | external (stdlib) | Timestamps on preferences |
| `strings` | external (stdlib) | Input normalization |
| `errors` | external (stdlib) | Error construction |
| `paths` | internal | OS-compliant path resolution |
| `repository` | internal | DTO types and `SettingsStore` interface |
| `logging` | internal | Injected logger for HTTP handler |
| `request` | internal | HTTP request parsing |
| `response` | internal | HTTP response helpers |

## Logging

- `SettingsHandler`: `logging.GetGlobal().WithComponent("handler-settings")`
- Manager does not use structured logging (operates before logger initialization during startup)

## Related

- [[Palimpsest-Writing-Software/architecture/settings/overview\|Settings System]]
- [[Palimpsest-Writing-Software/modules/settings-frontend/overview\|Settings Frontend Module]]
- [[Palimpsest-Writing-Software/api/settings/overview\|Settings API Service]]
