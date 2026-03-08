---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/architecture/logging/overview/","title":"Logging System Architecture","tags":["architecture","logging","infrastructure"],"updated":"2026-03-07T18:41:49.578-07:00"}
---


# Logging System Architecture

> [!NOTE] Unified, structured logging across all three application layers (Go backend, Svelte frontend, Tauri shell) with a shared configuration in `settings.json`, size-based file rotation, age-based retention cleanup, and sensitive field sanitization.

## Overview

Palimpsest's logging system provides structured, leveled logging for the entire application stack. All three runtime layers — Go backend, Svelte frontend, and Tauri desktop shell — read the same `LoggingConfig` block from `settings.json`, producing consistent, correlated log output.

Each layer writes to its own log file (per session or overwritten), stored in a shared log directory:

| Layer | Log File Pattern | Technology |
| ----- | ---------------- | ---------- |
| Go Backend | `backend-YYYY-MM-DD_HHMMSS.log` | `log/slog` with custom `rotatingFileWriter` |
| Svelte Frontend | `frontend-YYYY-MM-DD_HHMMSS.log` | Tauri IPC bridge (`log_frontend_message` command) |
| Tauri Shell | `tauri-YYYY-MM-DD_HHMMSS.log` | `tauri-plugin-log` with `RotationStrategy::KeepAll` |

When `overwrite: true` in settings, filenames collapse to `backend.log`, `frontend.log`, `tauri.log` (truncated each session).

## Shared Configuration

All layers consume the `logging` section of `settings.json`:

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `enabled` | bool | `true` | Master kill switch — `false` suppresses all output |
| `defaultLevel` | string | `"info"` | Base level: `"debug"`, `"info"`, `"warn"`, `"error"` |
| `format` | string | `"text"` | Output format: `"text"` (human-readable) or `"json"` (structured) |
| `output` | string | `"console"` | Output target: `"console"`, `"file"`, or `"both"` |
| `destination` | string | `""` | Custom log directory. Empty = OS-specific default |
| `overwrite` | bool | `false` | `true` = truncate on startup; `false` = timestamped per session |
| `addSource` | bool | `false` | Include source file:line in backend log entries |
| `retentionDays` | int | `30` | Auto-delete `.log` files older than N days. `0` = keep forever |
| `maxFileSizeMB` | int | `50` | Rotate file when it exceeds N MB. `0` = no limit |
| `components` | object | `{}` | Per-component level overrides (keyed by `"backend"`, `"frontend"`, `"tauri"`) |

### Component-Level Overrides

Each component can have its own log level that overrides `defaultLevel`:

```json
{
  "logging": {
    "defaultLevel": "info",
    "components": {
      "backend":  { "level": "debug" },
      "frontend": { "level": "warn" },
      "tauri":    { "level": "" }
    }
  }
}
```

An empty `level` string means "use `defaultLevel`". The Go backend further supports per-package granularity via the injected logger pattern (see below).

### Default Log Directory

| OS | Path |
| -- | ---- |
| Windows | `%APPDATA%\Palimpsest\logs\` |
| macOS | `~/Library/Application Support/Palimpsest/logs/` |
| Linux | `$XDG_CONFIG_HOME/palimpsest/logs/` or `~/.config/palimpsest/logs/` |

## Architecture by Layer

### Go Backend

The Go backend uses a **two-phase initialization** strategy:

1. **Phase 1 — Bootstrap** (`bootstrapLogger`): Before `settings.json` is available. Reads `LOG_LEVEL`, `LOG_FORMAT`, `LOG_SOURCE` from environment variables. Console output only (stderr). Follows 12-Factor: config comes from the environment.

2. **Phase 2 — Reconfigure** (`reconfigureLogger`): After the settings manager loads `settings.json` and applies env var overrides. Enables file output, rotation, retention cleanup. Replaces the global logger.

> [!IMPORTANT] All Go packages must use the **injected logger** pattern — never the global convenience functions. Struct-based packages add a `log *logging.Logger` field initialized via `logging.GetGlobal().WithComponent("<name>")`. Standalone functions use a package-level `var pkgLog = ...`. This ensures `slog` records the actual caller's source file instead of `logger.go`. See `CLAUDE.md` "Go Backend Logging" for details.

Key backend capabilities:

- **Size-based rotation** via `rotatingFileWriter` — thread-safe, auto-rotates when file exceeds `maxFileSizeMB`. Rotated files get numeric suffixes (`backend.1.log`, `backend.2.log`).
- **Retention cleanup** via `CleanupOldLogs` — runs once at startup after Phase 2, deletes `.log` files older than `retentionDays`.
- **Sensitive field sanitization** via `SanitizeValue`, `SanitizeHeaders`, `SanitizeQueryParams` — redacts fields matching patterns like `password`, `token`, `secret`, `authorization`, `cookie`, `api_key`, `credential`, `session`.
- **Structured error types** via `AppError` — maps domain errors to HTTP status codes with user-friendly messages, error codes, and optional detail maps.

### Svelte Frontend

The frontend logger (`loggerService.ts`) provides a `createLogger(component)` factory that returns a `Logger` interface with `debug`, `info`, `warn`, `error` methods.

Output routing:

- **Console output** — When `output` is `"console"` or `"both"`, uses `console[level](prefix, msg, data)`.
- **File output** — When `output` is `"file"` or `"both"` and running inside Tauri, sends structured `LogEntry` objects to the Go-style Tauri bridge via `invoke('log_frontend_message', { entry })`. Entries generated before the Tauri bridge loads are queued and flushed after `initLogger()` completes.
- **Global error handlers** — `installGlobalErrorHandlers()` captures `window.onerror` and `window.onunhandledrejection` at error level.

> [!TIP] The frontend logger has zero overhead when `enabled: false` — all methods become immediate no-ops after the threshold check.

### Tauri Shell

The Tauri layer uses `tauri-plugin-log` for its own framework logs (IPC, window events, plugin lifecycle). It reads the same `settings.json` and applies:

- Component-level override for `"tauri"` via `resolve_level("tauri")`
- Output targets matching the `output` setting (stdout, file folder, or both)
- The `log_frontend_message` Tauri command acts as the bridge between frontend `LogEntry` objects and the frontend log file on disk

## Cross-Layer Flow

```
settings.json
    │
    ├── Go Backend (main.go)
    │   ├── Phase 1: bootstrapLogger (env vars → console)
    │   └── Phase 2: reconfigureLogger (settings → file + console)
    │       ├── rotatingFileWriter → backend-*.log
    │       └── CleanupOldLogs → delete old .log files
    │
    ├── Svelte Frontend (loggerService.ts)
    │   ├── initLogger(config) → set levels, detect Tauri
    │   ├── console[level]() → browser console
    │   └── invoke('log_frontend_message') → Tauri IPC
    │       └── Tauri lib.rs → append to frontend-*.log
    │
    └── Tauri Shell (lib.rs)
        ├── tauri-plugin-log → tauri-*.log
        └── log_frontend_message command → frontend-*.log
```

## Design Decisions

| Decision | Rationale |
| -------- | --------- |
| Two-phase backend init | Structured logging available before settings.json loads; env vars always win (12-Factor) |
| Three separate log files | Each layer has different output format needs; simplifies debugging per-layer |
| Injected logger pattern | Fixes slog source attribution bug where all logs showed `logger.go` as caller |
| Global convenience functions removed | Enforces injected pattern; prevents accidental use |
| Silent degradation on file errors | Logging failures must never crash the application; disabled flag on rotatingFileWriter |
| Frontend entry queue | Logs generated during startup (before Tauri bridge loads) are not lost |
| Per-component overrides | Fine-grained control without restarting; keeps `defaultLevel` as baseline |

## Security

- **Sensitive field sanitization** — The `sanitize.go` module provides `SanitizeValue`, `SanitizeHeaders`, and `SanitizeQueryParams` functions that redact values for fields matching 11 sensitive patterns (password, token, secret, authorization, cookie, api_key, apikey, credential, session, passwd, key). All HTTP middleware and handlers should use these before logging request data.
- **Internal errors hidden from clients** — `InternalError()` wraps the underlying error but exposes only "an internal error occurred" to the client. The actual error is logged server-side.
- **No secrets in environment logging** — The bootstrap logger logs config keys (level, format, source) but never values of secrets.

## Performance

- **Zero-overhead disable** — Frontend logger checks `_enabled` flag first, short-circuiting all processing.
- **Level threshold check** — Both backend (`slog.HandlerOptions.Level`) and frontend (`LEVEL_PRIORITY` comparison) skip message formatting for below-threshold levels.
- **Mutex-protected file writer** — `rotatingFileWriter` uses a `sync.Mutex` per write. Under high concurrency, this could become a bottleneck; however, log writes are typically small and fast.
- **Fire-and-forget Tauri bridge** — Frontend file output is non-blocking (`Promise.catch(() => {})`) to avoid UI thread stalls.
- **Disabled file writer recovery** — If a file write or rotation fails, the writer sets `disabled = true` and silently drops subsequent writes rather than retrying or blocking.

## 12-Factor Compliance

- **Config via environment** — Phase 1 logger reads `LOG_LEVEL`, `LOG_FORMAT`, `LOG_SOURCE` from environment variables. After settings load, env var overrides are applied by the settings manager's `applyEnvOverrides()`, ensuring environment always wins over file-based config.
- **Treat logs as event streams** — Console output goes to stderr (standard stream); file output is an opt-in addition, not a replacement.
- **Dev/prod parity** — `DefaultConfig()` produces text/console for development; `ProductionConfig()` produces JSON/console with source. The same code path handles both.

## Diagrams


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Logging Initialization: Two-Phase Startup

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Log Output: Cross-Layer Routing

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Related

- [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend Module]]
- [[Palimpsest-Writing-Software/modules/logging-frontend/overview\|Logging Frontend Module]]
