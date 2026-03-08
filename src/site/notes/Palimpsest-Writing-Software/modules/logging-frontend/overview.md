---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-frontend/overview/","title":"Logging Frontend Module","tags":["module","logging-frontend"],"updated":"2026-03-05T07:11:17.450-07:00"}
---


# Logging Frontend

> [!NOTE] Svelte frontend structured logging service providing a `createLogger(component)` factory, configurable output routing (console, file via Tauri IPC bridge, or both), level-based filtering, global error/rejection capture, and startup entry queuing.

## Responsibilities

- Provide a `createLogger(component)` factory that returns a `Logger` interface with `debug`, `info`, `warn`, `error` methods
- Route log output to console, file (via Tauri IPC bridge), or both based on `settings.json` config
- Apply level-based filtering with per-component overrides (`frontend` component key)
- Support a master kill switch (`enabled: false`) with zero-overhead no-ops
- Queue log entries generated before the Tauri bridge loads and flush after initialization
- Capture uncaught JavaScript errors and unhandled Promise rejections via `installGlobalErrorHandlers()`

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `[[modules/logging-frontend/loggerService|LogEntry]]` | interface | Structured log entry forwarded to Tauri for file output |
| `[[modules/logging-frontend/loggerService|Logger]]` | interface | Component logger with debug/info/warn/error methods |
| `[[modules/logging-frontend/loggerService|initLogger]]` | function | Initialize the logger service from `LoggingConfig` |
| `[[modules/logging-frontend/loggerService|installGlobalErrorHandlers]]` | function | Install `window.onerror` and `onunhandledrejection` handlers |
| `[[modules/logging-frontend/loggerService|createLogger]]` | function | Create a named logger for a component |

## Internal Structure

**Logger Service** — Single-file implementation:
- `[[modules/logging-frontend/loggerService|loggerService.ts]]` — Initialization, factory, dispatch, Tauri bridge integration, global error handlers

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `LoggingConfig` from `systemSettings` | internal | Read-only logging configuration from settings.json |
| `@tauri-apps/api/core` (dynamic import) | external | `invoke()` for Tauri IPC bridge (file output) |

## Integration Points

- **Initialization** — `initLogger()` is called from `+layout.svelte` after `systemSettings.load()` completes. It receives the `LoggingConfig` object.
- **Global error capture** — `installGlobalErrorHandlers()` is called from `+layout.svelte` after `initLogger()`.
- **Component usage** — Any frontend module calls `createLogger('myComponent')` to get a logger. The component name appears as a prefix `[myComponent]` in console output and as the `component` field in file entries.
- **Tauri bridge** — When output includes `"file"`, the `logMessage()` function sends `LogEntry` objects to `log_frontend_message` Tauri command, which appends formatted lines to `frontend-*.log`.

## Logging

Uses its own `createLogger('loggerService')` for the initialization confirmation message. Pre-init log calls go to console regardless of output mode.

## Related

- [[Palimpsest-Writing-Software/architecture/logging/overview\|Logging System Architecture]]
- [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend Module]]
