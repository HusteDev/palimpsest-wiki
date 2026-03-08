---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/architecture/settings/overview/","title":"Settings System","tags":["architecture","settings","configuration","infrastructure"],"updated":"2026-03-07T18:41:44.967-07:00"}
---


# Settings System

> [!NOTE] Two-tier JSON file-based settings architecture separating read-only system configuration from mutable user preferences, with 12-Factor environment variable overrides, input validation with safe fallbacks, field migration for backwards compatibility, and a unified REST API that merges both tiers into a single DTO for frontend consumption.

## Overview

The Palimpsest settings system manages all application configuration through two distinct JSON files stored in the OS-appropriate configuration directory. The architecture enforces a strict separation between **system settings** (infrastructure, rarely changed) and **user preferences** (personal, frequently changed), while presenting a unified API to the frontend.

| Tier | File | Mutability | Scope |
| ---- | ---- | ---------- | ----- |
| **System Settings** | `ConfigDir/system/settings.json` | Read-only via API; edit file directly or use env vars | App environment, mode, port, database, logging, dev tools |
| **User Preferences** | `ConfigDir/system/preferences.json` | Read/write via API | Theme, font, panel layout, editor settings, word count, UI state |

## Directory Layout

Settings files follow OS conventions for configuration directories:

| OS | ConfigDir | DataDir |
| -- | --------- | ------- |
| **Windows** | `%APPDATA%\Palimpsest\` | `%USERPROFILE%\Documents\Palimpsest\` |
| **macOS** | `~/Library/Application Support/Palimpsest/` | `~/Documents/Palimpsest/` |
| **Linux** | `~/.config/palimpsest/` | `~/Documents/Palimpsest/` |

```
ConfigDir/
├── system/
│   ├── settings.json       ← System settings (infra config)
│   └── preferences.json    ← User preferences (personal config)
├── cache/
└── logs/
```

The `PALIMPSEST_ROOT` environment variable can override `ConfigDir`. The `PALIMPSEST_PROJECTS_DIR` environment variable can override the projects subdirectory within `DataDir`.

## System Settings

System settings control application infrastructure. They are loaded once at startup and can only be changed by editing `settings.json` directly (restart required) or via environment variables (12-Factor override).

| Field | Type | Default | Env Var | Description |
| ----- | ---- | ------- | ------- | ----------- |
| `app_env` | string | `"development"` | `APP_ENV` | Environment: `development` or `production` |
| `app_mode` | string | `"local"` | `APP_MODE` | Architecture: `local` (SQLite) or `online` (PostgreSQL, stub) |
| `port` | int | `8080` | `PORT` | HTTP server port (1–65535) |
| `data_directory` | string | OS DataDir | — | Root for projects, themes, plugins |
| `database_type` | string | `"sqlite"` | `DATABASE_TYPE` | Database engine: `sqlite` or `postgres` |
| `encryption_salt` | string | `""` | `ENCRYPTION_KEY_SALT` | Salt for local project password encryption |
| `dev_tools` | bool | `false` | `DEV_TOOLS` | Enable F12 console, debug endpoints, JSON viewer |
| `experimental_features` | bool | `false` | `EXPERIMENTAL_FEATURES` | Enable features still in development |
| `logging` | object | See [[Palimpsest-Writing-Software/architecture/logging/overview\|architecture/logging/overview]] | `LOG_*` (9 vars) | Full logging configuration for all layers |

### Validation

All system settings are validated after loading and after applying env overrides. Invalid values silently fall back to safe defaults — the application never fails to start due to bad configuration:

- Invalid `app_env` → `"development"`
- Invalid `app_mode` → `"local"`
- Port out of range → `8080`
- Invalid `database_type` → `"sqlite"`
- Invalid log levels → `"info"`, invalid format → `"text"`, invalid output → `"console"`
- Negative retention/size → `0`

## User Preferences

User preferences are mutable via the REST API and control the application's look, feel, and behavior.

### Theme & Accessibility

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `theme` | string | `"light"` | Active theme name (matches JSON theme file) |
| `font` | string | `"default"` | Font family: `"default"` or `"dyslexic"` (accessibility) |
| `locale` | string | `"en"` | UI language/locale |

### Editor Settings

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `spellcheck` | bool | `true` | Enable spell checking |
| `grammarCheck` | bool | `true` | Enable grammar checking |
| `autosave` | bool | `true` | Enable timed auto-save |
| `autosaveInterval` | int | `5` | Minutes between auto-saves (1–60) |
| `wordWrap` | bool | `true` | Enable word wrapping in editor |
| `saveOnNavigation` | string | `"prompt"` | Behavior on unsaved navigation: `"prompt"`, `"enable"`, `"disable"` |

### Word Count Display

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `showInTree` | bool | `true` | Show word counts in content navigation tree |
| `hierarchyVisibility` | Record | `{}` | Per content-type visibility overrides |
| `showWordCount` | bool | `true` | Show word count in status bar |
| `showCharCount` | bool | `true` | Show character count in status bar |
| `showPageEstimate` | bool | `false` | Show estimated page count in status bar |
| `charCountMode` | string | `"with-spaces"` | Character counting: `"with-spaces"` or `"without-spaces"` |
| `numberFormat` | string | `"locale"` | Number formatting: `"locale"`, `"plain"`, `"compact"` |
| `wordsPerPage` | int | `250` | Words per page for estimates (50–1000) |

### Panel Layout

Panel layout stores the full state of the workspace panel system — docked panel dimensions, visibility, component assignments, and floating panel instances. See [[Palimpsest-Writing-Software/features/panel-system/overview\|features/panel-system/overview]] for details.

### UI State

A dynamic `map[string]any` that persists hidden UI state across sessions:

| Key | Type | Description |
| --- | ---- | ----------- |
| `expandedTreeIds` | `string[]` | Content IDs of expanded nodes in the navigation tree |
| `collapsedPanels` | `{left, right, bottom}` | Boolean collapsed state for each panel position |
| `activeTab_left` | `string` | Active tab component ID for left panel |
| `activeTab_right` | `string` | Active tab component ID for right panel |
| Dynamic keys | `any` | Additional UI state from panel system and other components |

### Other Preferences

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `disableAutoNumbering` | bool | `false` | Disable automatic slug numbering for new content |
| `lastProjectSlug` | string | `""` | Last opened project for session restore |

## 12-Factor Compliance

The settings system follows 12-Factor App methodology for environment-based configuration:

1. **Config via environment** — Every system setting has a corresponding environment variable override. Environment variables always take precedence over file values.
2. **Strict separation** — Settings are separate from code; no hardcoded configuration values.
3. **Dev/prod parity** — `APP_ENV` controls environment-specific behavior (CORS, debug endpoints, logging verbosity).
4. **Safe defaults** — The application starts with sensible defaults even with zero configuration.
5. **No file modification from env** — Environment overrides are applied in-memory only; the JSON file on disk is never modified by env var values.

### Environment Variable Mapping

```
System:    APP_ENV, APP_MODE, PORT, DATABASE_TYPE, ENCRYPTION_KEY_SALT,
           DEV_TOOLS, EXPERIMENTAL_FEATURES
Logging:   LOG_ENABLED, LOG_LEVEL, LOG_FORMAT, LOG_OUTPUT, LOG_DESTINATION,
           LOG_OVERWRITE, LOG_SOURCE, LOG_RETENTION_DAYS, LOG_MAX_FILE_SIZE_MB
Paths:     PALIMPSEST_ROOT, PALIMPSEST_PROJECTS_DIR
```

Boolean env vars accept: `true`, `1`, `yes`, `on` / `false`, `0`, `no`, `off` (case-insensitive).

## Data Flow

### Startup (Load)

1. `paths.Resolve()` determines OS-appropriate `ConfigDir` and `DataDir`
2. `Manager.loadSystemSettings()` reads `settings.json` (creates with defaults if missing)
3. `Manager.applyEnvOverrides()` applies environment variable overrides in-memory
4. `validateSystemSettings()` and `validateLoggingConfig()` normalize invalid values
5. `Manager.loadUserPreferences()` reads `preferences.json` (creates with defaults if missing)
6. Field migration runs for backwards compatibility (addons → components, ms → minutes, etc.)
7. Manager is ready to serve API requests

### API Read (GET /api/settings)

1. Frontend calls `GET /api/settings`
2. Handler calls `Manager.GetSettings()`
3. Manager acquires read lock, builds `SettingsDTO` merging both tiers
4. DTO maps internal snake_case fields to frontend camelCase via JSON tags
5. Frontend `systemSettings.load()` and `preferences.load()` both consume this single response
6. Data is split: system fields → `systemSettings` store (read-only), user fields → `userPreferences` store (mutable)

### API Write (PUT /api/settings)

1. Frontend calls `PUT /api/settings` with partial update body
2. Handler parses `UpdateSettingsInput` — nil fields are skipped
3. Manager acquires write lock
4. Validation runs per-field (theme whitelist, font enum, autosave range, directory writability)
5. Only changed tiers are persisted (system → `settings.json`, user → `preferences.json`)
6. Updated `SettingsDTO` returned to frontend

## Field Migration

The backend handles backwards compatibility for preferences saved by older versions:

| Migration | Trigger | Action |
| --------- | ------- | ------ |
| Panel addons → components | `components` empty, `addons` non-empty | Copy `addons` to `components`, clear `addons` |
| Autosave interval ms → min | `autosaveInterval > 60` | Divide by 1000, min 1, fallback 5 |
| Missing `saveOnNavigation` | Empty string | Default to `"prompt"` |
| Missing `wordCount` | `wordsPerPage == 0` | Initialize with full defaults |
| Missing `uiState` | `nil` | Initialize with empty tree + collapsed panels |
| Missing `collapsedPanels` | `nil` in uiState | Add default `{left: false, right: false, bottom: false}` |
| Missing logging section | All logging fields zero-valued | Initialize with `defaultLoggingConfig()` |
| Missing `floating` in layout | Not an array | Default to empty array `[]` |

The frontend (`preferences.ts`) also performs its own migration layer using `migratePanelLayout()` for the same addons → components migration, ensuring robustness regardless of which side loads first.

## Security

- **Encryption salt protection** — The encryption salt is stored in settings but should be treated as sensitive. It is included in the combined DTO for system info purposes but is never sent to the frontend in production.
- **Directory validation** — `ValidateDirectory()` expands `~`, cleans paths, checks existence, verifies writability by creating a temp file, and rejects empty paths.
- **Input sanitization** — All string inputs are trimmed. Theme names are validated against a whitelist. Font must be `"default"` or `"dyslexic"`. SaveOnNavigation must be one of three known values.
- **Concurrent access** — `sync.RWMutex` protects all read and write operations on the in-memory settings state.
- **File permissions** — Settings files are written with `0o644` permissions (owner read/write, group/other read).

## Performance

- **In-memory caching** — Settings are loaded once at startup and cached in the `Manager` struct. All reads serve from memory (no file I/O per request).
- **Selective persistence** — `UpdateSettings` tracks which tier changed and only writes the affected JSON file.
- **Debounced frontend saves** — Panel layout and UI state updates use `PANEL_DEBOUNCE` (configurable) to batch rapid changes into a single API call, preventing excessive writes during drag-resize operations.
- **RWMutex** — Concurrent reads are non-blocking; only writes acquire an exclusive lock.
- **z-index normalization** — Floating panel z-indices are automatically normalized when they approach the modal z-index threshold (900), preventing overflow without visible user impact.

## Design Decisions

| Decision | Rationale |
| -------- | --------- |
| Two separate JSON files | System and user settings have different lifecycles — system rarely changes, user changes frequently. Separate files prevent accidental system config changes via the preferences API. |
| Unified API response | Frontend gets one `GET /api/settings` response containing both tiers, simplifying the API surface while maintaining backend separation. |
| Env vars override file, not modify | 12-Factor compliance: env vars are ephemeral; persisting them to disk would create confusion about the source of truth. |
| Silent fallback on invalid values | The application should always start, even with corrupt settings. Invalid values are normalized to safe defaults rather than causing startup failures. |
| Both-side migration | Both Go backend and TypeScript frontend handle field migration independently, ensuring correctness regardless of load order or API failures. |
| Debounced panel saves | Panel resize operations can fire dozens of updates per second during drag — debouncing collapses them into one API call. |
| Dynamic UIState map | Using `map[string]any` for UI state allows the frontend to persist arbitrary state (expanded sections, active tabs) without requiring backend schema changes. |

## Diagrams


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Settings Load: Startup Initialization

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Settings Update: Frontend Save to Disk

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Related

- [[Palimpsest-Writing-Software/modules/settings-backend/overview\|Settings Backend Module]]
- [[Palimpsest-Writing-Software/modules/settings-frontend/overview\|Settings Frontend Module]]
- [[Palimpsest-Writing-Software/api/settings/overview\|Settings API Service]]
- [[Palimpsest-Writing-Software/architecture/logging/overview\|Logging System]]
- [[Palimpsest-Writing-Software/features/panel-system/overview\|Panel System]]
