---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/settings-backend/settings-handler/","title":"settings.go","tags":["file","settings-backend","go","handler"],"updated":"2026-03-05T08:59:55.409-07:00"}
---


# `settings.go`

**Module:** `[[modules/settings-backend/overview|Settings Backend]]`
**Path:** `backend/internal/http/handlers/settings.go`

> [!NOTE] HTTP handler providing REST endpoints for reading, updating, and resetting application settings, plus system information and directory validation utilities.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SettingsHandler` | struct | HTTP handler for all settings-related endpoints |
| `NewSettingsHandler(settings)` | function | Constructor creating a handler with injected settings store and logger |

## SettingsHandler

### Properties

| Property | Type | Access | Description |
| -------- | ---- | ------ | ----------- |
| `settings` | `repository.SettingsStore` | private | Settings store interface for data access |
| `log` | `*logging.Logger` | private | Structured logger scoped to `"handler-settings"` component |

### Constructor

```go
func NewSettingsHandler(settings repository.SettingsStore) *SettingsHandler
```

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `settings` | `repository.SettingsStore` | yes | Settings store implementation (typically `*settings.Manager`) |

Returns a `*SettingsHandler` with an injected logger initialized via `logging.GetGlobal().WithComponent("handler-settings")`.

## Handler Methods

### `Settings(w, r)` -- Method Router

**Route:** `/api/settings`
**Methods:** GET, PUT

Routes requests by HTTP method:

| Method | Delegates To |
| ------ | ------------ |
| `GET` | `GetSettings(w, r)` |
| `PUT` | `UpdateSettings(w, r)` |
| Other | `response.MethodNotAllowed(w, r)` |

---

### `GetSettings(w, r)`

**Route:** `GET /api/settings`
**Auth required:** No

Returns the merged system + user settings as a single `SettingsDTO`.

**Response:**

| Status | Condition | Body |
| ------ | --------- | ---- |
| `200` | Success | `SettingsDTO` JSON |
| `500` | Store read failure | `{ "error": "failed to get settings" }` |

**Behavior:**

1. Calls `h.settings.GetSettings()` to retrieve the combined settings DTO.
2. Returns the DTO as a `200 OK` JSON response.
3. On error, returns `500 Internal Server Error`.

---

### `UpdateSettings(w, r)`

**Route:** `PUT /api/settings`
**Auth required:** No

Applies a partial update to settings. Only non-nil fields in the request body are modified.

**Request Body:** `repository.UpdateSettingsInput` (JSON). All fields are optional (pointer types).

**Response:**

| Status | Condition | Body |
| ------ | --------- | ---- |
| `200` | Success | Updated `SettingsDTO` JSON |
| `400` | Missing `Content-Type: application/json` | `{ "error": "content-type must be application/json" }` |
| `400` | Malformed JSON body | `{ "error": "invalid request body" }` |
| `400` | Validation failure (theme, font, interval, directory) | `{ "error": "<validation message>" }` |

**Behavior:**

1. Validates that `Content-Type` is `application/json`.
2. Parses the request body into `repository.UpdateSettingsInput` using `request.ParseSafely`.
3. Delegates to `h.settings.UpdateSettings(req)` which validates and persists changes.
4. Returns the updated settings DTO as `200 OK`.
5. Logs the update attempt at INFO level and the result at DEBUG level.

---

### `ResetSettings(w, r)`

**Route:** `POST /api/settings/reset`
**Auth required:** No

Resets user preferences to factory defaults. System settings (data directory, dev tools, logging) are not affected.

**Response:**

| Status | Condition | Body |
| ------ | --------- | ---- |
| `200` | Success | Reset `SettingsDTO` JSON |
| `405` | Not POST | Method Not Allowed |
| `500` | Store reset failure | `{ "error": "failed to reset settings" }` |

**Behavior:**

1. Validates the HTTP method is POST.
2. Delegates to `h.settings.ResetSettings()`.
3. Returns the reset settings DTO as `200 OK`.

---

### `GetSystemInfo(w, r)`

**Route:** `GET /api/settings/system`
**Auth required:** No

Returns system-wide information including app environment, mode, port, data directory, database type, logging config, and project count.

**Response:**

| Status | Condition | Body |
| ------ | --------- | ---- |
| `200` | Success | `SystemInfoDTO` JSON |
| `405` | Not GET | Method Not Allowed |
| `500` | Store read failure | `{ "error": "failed to get system info" }` |

**Behavior:**

1. Validates the HTTP method is GET.
2. Delegates to `h.settings.GetSystemInfo()` which scans the filesystem for project count.
3. Returns the system info DTO as `200 OK`.

---

### `ValidateDirectory(w, r)`

**Route:** `POST /api/settings/validate-directory`
**Auth required:** No

Validates that a directory path exists (or can be created), is writable, and returns the expanded absolute path. Supports `~` expansion.

**Request Body:**

```json
{
  "path": "/some/directory/path"
}
```

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `path` | `string` | yes | Directory path to validate (supports `~` expansion) |

**Response:**

| Status | Condition | Body |
| ------ | --------- | ---- |
| `200` | Valid and writable | `{ "expandedPath": "<abs_path>", "message": "directory is valid and writable" }` |
| `400` | Missing `Content-Type` | `{ "error": "content-type must be application/json" }` |
| `400` | Malformed body | `{ "error": "invalid request body" }` |
| `400` | Empty path | `{ "error": "path is required" }` |
| `400` | Invalid/unwritable path | `{ "error": "<validation message>" }` |
| `405` | Not POST | Method Not Allowed |

**Behavior:**

1. Validates HTTP method is POST and Content-Type is `application/json`.
2. Parses request body for the `path` field.
3. Validates path is non-empty.
4. Delegates to `h.settings.ValidateDirectory(req.Path)` which expands `~`, cleans the path, creates the directory, and tests write access.
5. Returns the expanded path and a success message.

---

### `GetProjectsDirectory(w, r)`

**Route:** `GET /api/settings/projects-directory`
**Auth required:** No

Returns the resolved absolute path to the projects directory (`DataDir/Projects`).

**Response:**

| Status | Condition | Body |
| ------ | --------- | ---- |
| `200` | Success | `{ "projectsDirectory": "<abs_path>" }` |
| `405` | Not GET | Method Not Allowed |
| `500` | Store read failure | `{ "error": "failed to get projects directory" }` |

**Behavior:**

1. Validates HTTP method is GET.
2. Delegates to `h.settings.GetProjectsDirectory()`.
3. Returns the directory path as `200 OK`.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `net/http` | stdlib | HTTP types (`ResponseWriter`, `Request`) |
| `request` | `internal/http/request` | Content-Type validation and safe body parsing |
| `response` | `internal/http/response` | Standardized HTTP response helpers (OK, BadRequest, InternalError, MethodNotAllowed) |
| `logging` | `internal/logging` | Structured logger with component scoping |
| `repository` | `internal/repository` | `SettingsStore` interface, `UpdateSettingsInput` DTO |

## Side Effects

None. The handler is stateless -- all state management is delegated to the injected `repository.SettingsStore` implementation. The handler only performs HTTP request/response translation and logging.

## Notes

> [!WARNING] The handler does not perform authentication or authorization checks. All endpoints are open. Auth will be required for the Pro/online mode (Release 2.0).

> [!WARNING] `UpdateSettings` returns `400 Bad Request` for validation failures (not `422 Unprocessable Entity`). This is consistent across all Palimpsest handlers.
