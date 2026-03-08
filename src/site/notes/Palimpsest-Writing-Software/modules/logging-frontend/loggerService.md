---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-frontend/logger-service/","title":"loggerService.ts","tags":["file","logging-frontend","service"],"updated":"2026-03-05T07:13:29.411-07:00"}
---


# `loggerService.ts`

**Module:** [[Palimpsest-Writing-Software/modules/logging-frontend/overview\|Logging Frontend]]
**Path:** `src/lib/services/loggerService.ts`

> [!NOTE] Frontend structured logging service that replaces raw `console.*` calls with a unified API. Provides configurable level filtering, dual output routing (console and/or file via Tauri IPC), startup entry queuing, and global error capture.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `LogEntry` | interface | Structured log entry forwarded to Tauri for file output |
| `Logger` | interface | Component logger with `debug`, `info`, `warn`, `error` methods |
| `initLogger(config)` | function | Initialize the logger service from `LoggingConfig` |
| `installGlobalErrorHandlers()` | function | Install `window.onerror` and `onunhandledrejection` handlers |
| `createLogger(component)` | function | Create a named logger for a component |

## `LogEntry` Interface

Structured log entry created by `logMessage()` and forwarded to the Tauri backend via the `log_frontend_message` command.

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `timestamp` | `string` | yes | ISO 8601 timestamp (`new Date().toISOString()`) |
| `level` | `'debug' \| 'info' \| 'warn' \| 'error'` | yes | Severity level |
| `component` | `string` | yes | Name of the component that generated this entry |
| `message` | `string` | yes | Human-readable log message |
| `data` | `Record<string, unknown>` | no | Optional structured data payload |

## `Logger` Interface

Returned by `createLogger()`. Each method delegates to the internal `logMessage()` dispatch function.

| Method | Parameters | Returns | Description |
| ------ | ---------- | ------- | ----------- |
| `debug(msg, data?)` | `msg: string`, `data?: Record<string, unknown>` | `void` | Debug-level log (development diagnostics) |
| `info(msg, data?)` | `msg: string`, `data?: Record<string, unknown>` | `void` | Info-level log (normal operations) |
| `warn(msg, data?)` | `msg: string`, `data?: Record<string, unknown>` | `void` | Warning-level log (unexpected but recoverable) |
| `error(msg, data?)` | `msg: string`, `data?: Record<string, unknown>` | `void` | Error-level log (failures requiring attention) |

## `initLogger` Behavior

```
initLogger(config: LoggingConfig) → Promise<void>
```

Called once from `+layout.svelte` after `systemSettings` are loaded. Configures all module-level state and prepares the Tauri bridge for file output.

**Steps:**

1. **Guard** — If `config` is missing or not an object, logs a raw `console.warn` and returns early (logger may be in an invalid state).
2. **Enable flag** — Sets `_enabled` from `config.enabled`. Defaults to `true` if not explicitly `false`.
3. **Level resolution** — Resolves the frontend component level: `config.components?.frontend?.level`, falling back to `config.defaultLevel`, then `'info'`. Converts to numeric threshold via `LEVEL_PRIORITY`.
4. **Output targets** — Reads `config.output` (default `'console'`). Sets `_outputIncludesConsole` and `_outputIncludesFile` flags based on whether the value is `"console"`, `"file"`, or `"both"`.
5. **Tauri detection** — Checks for `window.__TAURI__` or `window.__TAURI_INTERNALS__` to determine if the Tauri bridge is available.
6. **Dynamic import** — If file output is needed and Tauri is detected, awaits `loadTauriInvoke()` to cache the `invoke` function.
7. **Marks `_initialized = true`**.
8. **Flush queue** — If `_tauriInvoke` is ready and `_pendingEntries` has items, sends each entry via `log_frontend_message`. Clears the queue.
9. **Self-test** — Creates a `'loggerService'` logger and emits an `info` entry with the resolved configuration (enabled, level, threshold, output, tauriBridge, tauriInvokeReady).

## `installGlobalErrorHandlers` Behavior

```
installGlobalErrorHandlers() → void
```

Called once from `+layout.svelte` after `initLogger()`. No-ops if `window` is undefined (SSR guard).

**Handlers installed:**

| Handler | Captures | Data fields |
| ------- | -------- | ----------- |
| `window.onerror` | Uncaught JavaScript errors | `message`, `source`, `line`, `column`, `stack` |
| `window.onunhandledrejection` | Unhandled Promise rejections | `reason`, `stack` |

Both handlers log at the `error` level through a `createLogger('global')` instance.

## `createLogger` Behavior

```
createLogger(component: string) → Logger
```

Returns a `Logger` object whose `debug`, `info`, `warn`, and `error` methods each delegate to `logMessage()` with the given `component` name. The component name appears as:
- A `[component]` prefix in console output
- The `component` field in `LogEntry` objects sent to file output

When logging is disabled (`_enabled = false`), all methods are immediate no-ops because `logMessage()` short-circuits on the first line.

## Internal Dispatch — `logMessage`

```
logMessage(level, component, msg, data?) → void
```

Core routing function. Every `Logger` method calls this.

**Flow:**

1. **Kill switch** — Returns immediately if `!_enabled` (zero overhead).
2. **Level gate** — Returns if the numeric priority of `level` is below `_levelThreshold`.
3. **Console output** — If `_outputIncludesConsole` is true, or if `!_initialized` (pre-init fallback), writes to `console[level]` with a `[component]` prefix. Omits the `data` argument when it is `undefined` or `null`.
4. **File output** — If `_outputIncludesFile` is true:
   - Builds a `LogEntry` with an ISO timestamp.
   - If `_tauriInvoke` is ready: sends fire-and-forget via `_tauriInvoke('log_frontend_message', { entry })`. Errors are silently caught.
   - If `!_initialized`: pushes the entry onto `_pendingEntries` for later flush.
   - If `_initialized` but `_tauriInvoke` is null: bridge failed to load; entry is dropped from file output (console may still have it).
5. **STUB** — Comment marks where WebApp/Pro mode would send entries to a cloud logging endpoint.

## Tauri Bridge Integration

The Tauri IPC bridge is used to forward `LogEntry` objects to the Go backend for file output.

**Dynamic import pattern:**

`loadTauriInvoke()` dynamically imports `@tauri-apps/api/core` to obtain the `invoke` function. This avoids a hard dependency on the Tauri API, which would break in non-Tauri contexts (browser dev server, future WebApp mode).

| State | Behavior |
| ----- | -------- |
| Import succeeds | `_tauriInvoke` is cached; entries are sent immediately via `log_frontend_message` |
| Import fails | `_tauriBridgeAvailable` set to `false`; `_tauriInvoke` stays `null`; file output is silently disabled |

**Entry queuing:**

Log entries generated before `initLogger()` completes (while `_initialized` is `false`) are pushed onto `_pendingEntries`. After initialization, the queue is flushed to the Tauri bridge and cleared. This prevents log loss during the startup race between component initialization and logger setup.

## Internal State (Module-Level)

| Variable | Type | Default | Purpose |
| -------- | ---- | ------- | ------- |
| `_enabled` | `boolean` | `true` | Master kill switch; when `false`, all log methods are no-ops |
| `_levelThreshold` | `number` | `1` (info) | Minimum numeric priority to output |
| `_tauriBridgeAvailable` | `boolean` | `false` | Whether Tauri environment was detected |
| `_outputIncludesFile` | `boolean` | `false` | Whether file output is configured |
| `_outputIncludesConsole` | `boolean` | `true` | Whether console output is configured |
| `_tauriInvoke` | `function \| null` | `null` | Cached Tauri `invoke()` function |
| `_pendingEntries` | `LogEntry[]` | `[]` | Queue for entries before bridge ready |
| `_initialized` | `boolean` | `false` | Whether `initLogger` has completed |
| `LEVEL_PRIORITY` | `Record<string, number>` | `{ debug: 0, info: 1, warn: 2, error: 3 }` | Numeric priority map for level comparison |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `LoggingConfig` | `$lib/stores/systemSettings` | Type for the logging configuration from `settings.json` |
| `invoke` | `@tauri-apps/api/core` (dynamic) | Tauri IPC bridge for file output |

## Side Effects

- Reads `window.__TAURI__` and `window.__TAURI_INTERNALS__` during initialization to detect the Tauri environment
- Sets `window.onerror` and `window.onunhandledrejection` when `installGlobalErrorHandlers()` is called
- Module-level mutable state (`_enabled`, `_levelThreshold`, `_tauriInvoke`, etc.) is shared across all callers within the same JS context
- Pre-init log entries always go to console regardless of output mode configuration
