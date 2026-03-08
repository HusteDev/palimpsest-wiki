---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/settings/overview/","title":"Settings API","tags":["api","settings","rest"],"updated":"2026-03-05T08:58:32.423-07:00"}
---


# Settings API

> [!NOTE] REST API service for reading and writing application settings, including system configuration, user preferences, directory validation, and system information.

## Base Path

```
/api/settings
```

## Endpoints

| Method | Path                                | Description                              | Auth |
| ------ | ----------------------------------- | ---------------------------------------- | ---- |
| GET    | `/api/settings`                     | Get combined system + user settings      | None |
| PUT    | `/api/settings`                     | Partial update of settings               | None |
| POST   | `/api/settings/reset`               | Reset user preferences to defaults       | None |
| GET    | `/api/settings/system`              | Get system information                   | None |
| POST   | `/api/settings/validate-directory`  | Validate a directory path                | None |
| GET    | `/api/settings/projects-directory`  | Get the resolved projects directory path | None |

---

## GET `/api/settings`

**Handler:** `[[modules/settings-backend/overview|SettingsHandler.GetSettings]]`

Returns the combined system settings and user preferences as a single flat SettingsDTO.

### Request

**Auth required:** no

No path parameters, query parameters, or request body.

### Response

#### Success — `200 OK`

The response merges system-level fields with user preference fields into a single object.

**System fields (read-only):**

| Field                  | Type               | Description                                                |
| ---------------------- | ------------------ | ---------------------------------------------------------- |
| `appEnv`               | `string`           | Application environment (`"development"` or `"production"`) |
| `appMode`              | `string`           | Application architecture (`"local"` or `"online"`)         |
| `port`                 | `int`              | HTTP server port for the backend API                       |
| `dataDirectory`        | `string`           | Root directory for projects, themes, and plugins           |
| `databaseType`         | `string`           | Database engine (`"sqlite"` or `"postgres"`)               |
| `encryptionSalt`       | `string`           | Salt for local project password encryption                 |
| `devTools`             | `bool`             | Enables debug features (F12 console, debug messages, etc.) |
| `experimentalFeatures` | `bool`             | Enables features still in development                      |
| `logging`              | `LoggingConfigDTO` | Full logging configuration (read-only via API)             |

**User preference fields:**

| Field                  | Type                  | Description                                             |
| ---------------------- | --------------------- | ------------------------------------------------------- |
| `theme`                | `string`              | Active theme identifier                                 |
| `font`                 | `string`              | Editor font family                                      |
| `locale`               | `string`              | User locale string                                      |
| `panelLayout`          | `PanelLayoutDTO`      | Layout configuration for all UI panels                  |
| `editor`               | `EditorSettingsDTO`   | Editor-specific preferences                             |
| `wordCount`            | `WordCountSettingsDTO`| Word count display preferences                          |
| `disableAutoNumbering` | `bool`                | Whether automatic content numbering is disabled         |
| `lastProjectSlug`      | `string`              | Slug of the most recently opened project                |
| `uiState`              | `UIStateDTO`          | Hidden UI state persisted across sessions (dynamic map) |
| `createdAt`            | `string (ISO 8601)`   | Timestamp when preferences were created                 |
| `updatedAt`            | `string (ISO 8601)`   | Timestamp of last preferences update                    |

#### Error Responses

| Status | Condition      | Body                                    |
| ------ | -------------- | --------------------------------------- |
| `500`  | Internal error | `{ "error": "failed to get settings" }` |

---

## PUT `/api/settings`

**Handler:** `[[modules/settings-backend/overview|SettingsHandler.UpdateSettings]]`

Partial update of settings. Uses `UpdateSettingsInput` where all fields are pointers. Fields set to `nil` (omitted from JSON) are skipped; only non-nil fields are applied.

### Request

**Auth required:** no
**Content-Type:** `application/json` (required)

#### Request Body

All fields are optional. Include only the fields you want to change.

**Updatable system settings:**

| Field           | Type     | Description                |
| --------------- | -------- | -------------------------- |
| `dataDirectory` | `string` | Root data directory path   |

**Updatable user preferences:**

| Field                  | Type                  | Description                                   |
| ---------------------- | --------------------- | --------------------------------------------- |
| `theme`                | `string`              | Theme identifier (validated against whitelist) |
| `font`                 | `string`              | Font family (validated against enum)           |
| `locale`               | `string`              | User locale string                             |
| `panelLayout`          | `PanelLayoutDTO`      | Full panel layout configuration                |
| `editor`               | `EditorSettingsDTO`   | Editor preferences                             |
| `wordCount`            | `WordCountSettingsDTO`| Word count preferences                         |
| `disableAutoNumbering` | `bool`                | Disable automatic content numbering            |
| `lastProjectSlug`      | `string`              | Slug of last opened project                    |
| `uiState`              | `UIStateDTO`          | UI state map                                   |

> [!IMPORTANT] Logging configuration is NOT updateable via the API. Edit `settings.json` directly and restart the application.

#### Validation Rules

| Field               | Rule                                              |
| ------------------- | ------------------------------------------------- |
| `theme`             | Must be in the allowed theme whitelist             |
| `font`              | Must match a valid font enum value                 |
| `editor.autosaveInterval` | Must be between 1 and 60 (minutes)           |
| `dataDirectory`     | Directory must exist and be writable               |
| `wordCount.charCountMode` | `"with-spaces"` or `"without-spaces"`        |
| `wordCount.numberFormat`  | `"locale"`, `"plain"`, or `"compact"`        |
| `wordCount.wordsPerPage`  | Must be between 50 and 1000                  |

### Response

#### Success — `200 OK`

Returns the full updated `SettingsDTO` (same schema as GET `/api/settings`).

#### Error Responses

| Status | Condition                         | Body                                     |
| ------ | --------------------------------- | ---------------------------------------- |
| `400`  | Missing `Content-Type` header     | `{ "error": "content-type must be application/json" }` |
| `400`  | Malformed JSON body               | `{ "error": "invalid request body" }`    |
| `400`  | Validation failure                | `{ "error": "<validation message>" }`    |
| `405`  | Wrong HTTP method                 | Method Not Allowed                       |

---

## POST `/api/settings/reset`

**Handler:** `[[modules/settings-backend/overview|SettingsHandler.ResetSettings]]`

Resets all user preferences back to their default values. System settings (data directory, logging, environment, etc.) are NOT affected.

### Request

**Auth required:** no

No request body required.

### Response

#### Success — `200 OK`

Returns the full `SettingsDTO` with user preference fields reset to defaults (same schema as GET `/api/settings`).

#### Error Responses

| Status | Condition      | Body                                      |
| ------ | -------------- | ----------------------------------------- |
| `405`  | Wrong method   | Method Not Allowed                        |
| `500`  | Internal error | `{ "error": "failed to reset settings" }` |

---

## GET `/api/settings/system`

**Handler:** `[[modules/settings-backend/overview|SettingsHandler.GetSystemInfo]]`

Returns system-level information including all system settings, a count of projects found on the filesystem, and the application version string.

### Request

**Auth required:** no

No path parameters, query parameters, or request body.

### Response

#### Success — `200 OK`

Returns a `SystemInfoDTO`:

| Field                  | Type               | Description                                                  |
| ---------------------- | ------------------ | ------------------------------------------------------------ |
| `appEnv`               | `string`           | Application environment (`"development"` or `"production"`)   |
| `appMode`              | `string`           | Application architecture (`"local"` or `"online"`)           |
| `port`                 | `int`              | HTTP server port                                             |
| `dataDirectory`        | `string`           | Root data directory path                                     |
| `databaseType`         | `string`           | Database engine (`"sqlite"` or `"postgres"`)                 |
| `encryptionSalt`       | `string`           | Encryption salt for local projects                           |
| `devTools`             | `bool`             | Debug features enabled                                       |
| `experimentalFeatures` | `bool`             | Experimental features enabled                                |
| `logging`              | `LoggingConfigDTO` | Full logging configuration                                   |
| `projectCount`         | `int`              | Number of projects found by scanning `.db` files in data dir |
| `version`              | `string`           | Application version string                                   |

#### Error Responses

| Status | Condition      | Body                                         |
| ------ | -------------- | -------------------------------------------- |
| `405`  | Wrong method   | Method Not Allowed                           |
| `500`  | Internal error | `{ "error": "failed to get system info" }`   |

---

## POST `/api/settings/validate-directory`

**Handler:** `[[modules/settings-backend/overview|SettingsHandler.ValidateDirectory]]`

Validates that a given directory path exists and is writable. Supports `~` (tilde) expansion for the user home directory. Useful for verifying a data directory before applying it.

### Request

**Auth required:** no
**Content-Type:** `application/json` (required)

#### Request Body

```json
{
  "path": "/absolute/path/to/directory"
}
```

| Field  | Type     | Required | Description                         |
| ------ | -------- | -------- | ----------------------------------- |
| `path` | `string` | yes      | Directory path to validate. Supports `~` expansion. |

### Response

#### Success — `200 OK`

```json
{
  "expandedPath": "/home/user/resolved/path",
  "message": "directory is valid and writable"
}
```

| Field          | Type     | Description                              |
| -------------- | -------- | ---------------------------------------- |
| `expandedPath` | `string` | The fully resolved absolute path         |
| `message`      | `string` | Confirmation message                     |

#### Error Responses

| Status | Condition                     | Body                                                       |
| ------ | ----------------------------- | ---------------------------------------------------------- |
| `400`  | Missing `Content-Type`        | `{ "error": "content-type must be application/json" }`     |
| `400`  | Malformed JSON body           | `{ "error": "invalid request body" }`                      |
| `400`  | Empty path                    | `{ "error": "path is required" }`                          |
| `400`  | Directory invalid or not writable | `{ "error": "<validation message>" }`                  |
| `405`  | Wrong method                  | Method Not Allowed                                         |

---

## GET `/api/settings/projects-directory`

**Handler:** `[[modules/settings-backend/overview|SettingsHandler.GetProjectsDirectory]]`

Returns the resolved absolute path to the projects directory.

### Request

**Auth required:** no

No path parameters, query parameters, or request body.

### Response

#### Success — `200 OK`

```json
{
  "projectsDirectory": "/home/user/Palimpsest/Projects"
}
```

| Field               | Type     | Description                         |
| ------------------- | -------- | ----------------------------------- |
| `projectsDirectory` | `string` | Resolved absolute path to projects  |

#### Error Responses

| Status | Condition      | Body                                               |
| ------ | -------------- | -------------------------------------------------- |
| `405`  | Wrong method   | Method Not Allowed                                 |
| `500`  | Internal error | `{ "error": "failed to get projects directory" }`  |

---

## Data Transfer Objects

### SettingsDTO

The combined response for GET and PUT `/api/settings`. Merges system-level fields with user preferences into a single flat object. See field tables in the GET endpoint above.

**Source:** `backend/internal/repository/settings.go`

### UpdateSettingsInput

Partial update input for PUT `/api/settings`. All fields are pointers (`*type`). Omitted fields (nil) are not modified.

**Source:** `backend/internal/repository/settings.go`

### SystemInfoDTO

Response for GET `/api/settings/system`. Extends system settings with `projectCount` and `version`.

**Source:** `backend/internal/repository/settings.go`

### Supporting DTOs

| DTO                        | Description                                           |
| -------------------------- | ----------------------------------------------------- |
| `LoggingConfigDTO`         | Full logging configuration (read-only via API)        |
| `ComponentLoggingDTO`      | Per-component log level override                      |
| `PanelLayoutDTO`           | Layout for left, right, bottom, and floating panels   |
| `PanelConfigDTO`           | Configuration for a single docked panel position      |
| `FloatingPanelInstanceDTO` | Position, size, and state of a floating panel          |
| `EditorSettingsDTO`        | Editor preferences (spellcheck, autosave, word wrap)  |
| `WordCountSettingsDTO`     | Word count display preferences                        |
| `UIStateDTO`               | Dynamic map of persisted UI state                     |

---

## Frontend Consumers

| Store                                      | Usage                                                        |
| ------------------------------------------ | ------------------------------------------------------------ |
| `[[modules/settings-frontend/overview|systemSettings.ts]]` | Calls GET `/api/settings` at startup to load system config  |
| `[[modules/settings-frontend/overview|preferences.ts]]`    | Calls GET `/api/settings` at startup, PUT `/api/settings` for updates |

---

## Backend Architecture

**Handler:** `SettingsHandler` (`backend/internal/http/handlers/settings.go`)
**Interface:** `SettingsStore` (`backend/internal/repository/settings.go`)

Settings are stored in JSON files on disk, not in per-project SQLite databases. The `SettingsStore` interface abstracts persistence so the implementation can change without affecting the handler layer.

The handler delegates all business logic (validation, merging, defaults) to the `SettingsStore` implementation. The handler is responsible only for HTTP concerns: method routing, content-type validation, request parsing, and response serialization.

---

## Related

- [[Palimpsest-Writing-Software/architecture/settings/overview\|Architecture: Settings]]
- [[Palimpsest-Writing-Software/modules/settings-backend/overview\|Module: Settings Backend]]
- [[Palimpsest-Writing-Software/modules/settings-frontend/overview\|Module: Settings Frontend]]
