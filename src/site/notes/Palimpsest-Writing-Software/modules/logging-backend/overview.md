---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-backend/overview/","title":"Logging Backend Module","tags":["module","logging-backend"],"updated":"2026-03-05T07:10:51.454-07:00"}
---


# Logging Backend

> [!NOTE] Go structured logging package providing leveled log output via `log/slog`, size-based file rotation, age-based retention cleanup, structured application error types with HTTP status mapping, and sensitive field sanitization for safe log output.

## Responsibilities

- Provide a `Logger` type wrapping `slog.Logger` with convenience methods (`WithComponent`, `WithRequestID`, `WithError`, `LogRequest`, `LogStartup`, `LogShutdown`)
- Manage output routing: console (stderr), file (rotating), or both (`io.MultiWriter`)
- Rotate log files when they exceed a configurable size limit (`rotatingFileWriter`)
- Clean up old log files based on a configurable retention period (`CleanupOldLogs`)
- Define structured application error types (`AppError`) with HTTP status codes and error codes
- Provide sentinel errors and typed constructors for common error conditions
- Redact sensitive field values (passwords, tokens, API keys) before they reach log output

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `[[modules/logging-backend/logger|Logger]]` | struct | Wraps `slog.Logger` with lifecycle management and convenience methods |
| `[[modules/logging-backend/logger|Config]]` | struct | Logger configuration (level, format, output, file path, rotation) |
| `[[modules/logging-backend/logger|New]]` | function | Create a Logger from Config (append mode) |
| `[[modules/logging-backend/logger|NewFileLogger]]` | function | Create a Logger with explicit overwrite control |
| `[[modules/logging-backend/logger|Default]]` | function | Create a development Logger (console, info, text) |
| `[[modules/logging-backend/logger|DefaultConfig]]` | function | Return development Config |
| `[[modules/logging-backend/logger|ProductionConfig]]` | function | Return production Config (JSON, source enabled) |
| `[[modules/logging-backend/logger|SetGlobal]]` | function | Set the package-level global Logger instance |
| `[[modules/logging-backend/logger|GetGlobal]]` | function | Get the current global Logger instance |
| `[[modules/logging-backend/retention|CleanupOldLogs]]` | function | Delete `.log` files older than N days |
| `[[modules/logging-backend/errors|AppError]]` | struct | Structured application error with HTTP status and code |
| `[[modules/logging-backend/errors|NotFoundError]]` | function | Create a 404 AppError |
| `[[modules/logging-backend/errors|AlreadyExistsError]]` | function | Create a 409 AppError |
| `[[modules/logging-backend/errors|ValidationError]]` | function | Create a 400 AppError |
| `[[modules/logging-backend/errors|InvalidInputError]]` | function | Create a 400 AppError |
| `[[modules/logging-backend/errors|UnauthorizedError]]` | function | Create a 401 AppError |
| `[[modules/logging-backend/errors|ForbiddenError]]` | function | Create a 403 AppError |
| `[[modules/logging-backend/errors|InternalError]]` | function | Create a 500 AppError (hides underlying error from client) |
| `[[modules/logging-backend/errors|DatabaseError]]` | function | Create a 500 AppError for database failures |
| `[[modules/logging-backend/errors|WrapError]]` | function | Wrap an error with additional context message |
| `[[modules/logging-backend/errors|GetHTTPStatus]]` | function | Extract HTTP status from any error (500 default) |
| `[[modules/logging-backend/errors|GetErrorCode]]` | function | Extract error code from any error |
| `[[modules/logging-backend/errors|GetUserMessage]]` | function | Extract user-friendly message from any error |
| `[[modules/logging-backend/sanitize|SanitizeValue]]` | function | Redact a value if its field name matches a sensitive pattern |
| `[[modules/logging-backend/sanitize|SanitizeHeaders]]` | function | Redact sensitive HTTP headers |
| `[[modules/logging-backend/sanitize|SanitizeQueryParams]]` | function | Redact sensitive URL query parameters |

## Internal Structure

**Core Logger** — The main logger implementation with file rotation:
- `[[modules/logging-backend/logger|logger.go]]` — `Logger` struct, `Config`, constructors, `rotatingFileWriter`, global instance management

**Retention** — Startup cleanup of old log files:
- `[[modules/logging-backend/retention|retention.go]]` — `CleanupOldLogs` function

**Error Types** — Structured application errors with HTTP mapping:
- `[[modules/logging-backend/errors|errors.go]]` — `AppError` struct, sentinel errors, typed constructors, error inspection utilities

**Sanitization** — Sensitive field redaction:
- `[[modules/logging-backend/sanitize|sanitize.go]]` — `SanitizeValue`, `SanitizeHeaders`, `SanitizeQueryParams`

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `log/slog` | external (stdlib) | Structured, leveled logging |
| `io` | external (stdlib) | `MultiWriter` for dual output, `Closer` for file lifecycle |
| `os` | external (stdlib) | File operations, stderr output |
| `sync` | external (stdlib) | `Mutex` for thread-safe file writer |
| `net/http` | external (stdlib) | HTTP status codes for `AppError` |
| `net/url` | external (stdlib) | `url.Values` for query param sanitization |

## Logging

> [!WARNING] The `logging` package itself cannot use the injected logger pattern for its own internal errors (e.g., rotation failures, file write errors) because it would create a circular dependency. These critical errors are written directly to `os.Stderr` with `[logging]` prefix.

## Related

- [[Palimpsest-Writing-Software/architecture/logging/overview\|Logging System Architecture]]
- [[Palimpsest-Writing-Software/modules/logging-frontend/overview\|Logging Frontend Module]]
