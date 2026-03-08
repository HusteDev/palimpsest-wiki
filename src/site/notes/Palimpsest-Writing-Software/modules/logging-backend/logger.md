---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-backend/logger/","title":"logger.go","tags":["file","logging-backend"],"updated":"2026-03-05T07:13:05.415-07:00"}
---


# `logger.go`

**Module:** [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend]]
**Path:** `backend/internal/logging/logger.go`

> [!NOTE] Core logging infrastructure: logger creation, configuration, file rotation, global instance management, and structured convenience methods.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Level` | type alias | Alias for `slog.Level`; represents a logging severity level |
| `LevelDebug` | const | Debug severity (maps to `slog.LevelDebug`) |
| `LevelInfo` | const | Info severity (maps to `slog.LevelInfo`) |
| `LevelWarn` | const | Warn severity (maps to `slog.LevelWarn`) |
| `LevelError` | const | Error severity (maps to `slog.LevelError`) |
| `Config` | struct | Logger configuration: level, format, output writer, source toggle, file path, max file size |
| `Logger` | struct | Wraps `*slog.Logger` with lifecycle management and convenience methods |
| `DefaultConfig()` | func | Returns development defaults (info level, text format, stderr) |
| `ProductionConfig()` | func | Returns production defaults (info level, JSON format, stderr, source enabled) |
| `New(cfg Config)` | func | Creates a `*Logger` from a `Config`. File output uses append mode. Falls back to console on file error. |
| `NewFileLogger(cfg Config, overwrite bool)` | func | Creates a `*Logger` with explicit file overwrite control. Requires non-empty `FilePath`. |
| `Default()` | func | Shortcut for `New(DefaultConfig())` |
| `SetGlobal(l *Logger)` | func | Sets the package-level global logger and calls `slog.SetDefault` |
| `GetGlobal()` | func | Returns the current global `*Logger` |

### Logger Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `Close()` | `error` | Closes the underlying file handle if any; no-op for console-only loggers |
| `With(args ...any)` | `*Logger` | Returns a new logger with additional structured attributes |
| `WithComponent(name string)` | `*Logger` | Adds a `"component"` attribute identifying the subsystem |
| `WithRequestID(id string)` | `*Logger` | Adds a `"request_id"` attribute for request tracing |
| `WithError(err error)` | `*Logger` | Adds an `"error"` attribute with the error message |
| `LogRequest(ctx, method, path, status, duration)` | -- | Logs an HTTP request at info level with method, path, status, and duration_ms |
| `LogStartup(version, goVersion string)` | -- | Logs application startup with version, Go version, OS, and architecture |
| `LogShutdown(reason string)` | -- | Logs the shutdown reason |

## Config Struct

```go
type Config struct {
    Level         Level      // Minimum log level (LevelDebug, LevelInfo, LevelWarn, LevelError)
    Format        string     // "text" (human-readable) or "json" (structured)
    Output        io.Writer  // Console output destination; defaults to os.Stderr if nil
    AddSource     bool       // Include source file and line number in log entries
    FilePath      string     // Path to log file; empty = no file output
    MaxFileSizeMB int        // Max file size in MB before rotation; 0 = no limit
}
```

`DefaultConfig()` returns: Level=Info, Format="text", Output=os.Stderr, AddSource=false, no file output.

`ProductionConfig()` returns: Level=Info, Format="json", Output=os.Stderr, AddSource=true, no file output.

## Logger Struct

```go
type Logger struct {
    *slog.Logger          // Embedded standard structured logger
    closer io.Closer      // File writer handle for cleanup (nil for console-only)
}
```

The `Logger` embeds `*slog.Logger` so all standard `slog` methods (Debug, Info, Warn, Error, and their `*Context` variants) are available directly.

## File Output and Rotation

When `Config.FilePath` is set, the logger writes to a file via an internal `rotatingFileWriter`. Output routing:

- **FilePath set + Output set:** writes to both via `io.MultiWriter` ("both" mode)
- **FilePath set + Output nil/Discard:** writes only to file
- **FilePath empty:** writes to Output (or stderr if nil)

### rotatingFileWriter (unexported)

```go
type rotatingFileWriter struct {
    filePath      string       // Base path (e.g., /path/to/backend.log)
    maxBytes      int64        // Size limit before rotation; 0 = no limit
    currentFile   *os.File     // Active file handle
    currentSize   int64        // Tracks current file size in bytes
    rotationCount int          // Number of rotations performed
    disabled      bool         // True if a write/rotate error occurred
    mu            sync.Mutex   // Protects all mutable state
}
```

**Key behaviors:**

- `newRotatingFileWriter(filePath, maxSizeMB, overwrite)` -- Creates parent directories via `os.MkdirAll`. Opens file in append or truncate mode based on the `overwrite` flag.
- `Write(p []byte)` -- Thread-safe. Checks if rotation is needed before writing. On any error (rotation failure or write failure), sets `disabled=true` and silently drops all future writes to prevent error cascading. Does not propagate errors to the slog handler.
- `Close()` -- Closes the file handle under the mutex.
- `rotateLocked()` -- Renames the current file to `backend.N.log` (incrementing N), then opens a fresh file at the original path. Must be called with the mutex held.

**Rotation naming scheme:** `backend.log` becomes `backend.1.log`, `backend.2.log`, etc.

## Global Logger

```go
var global = Default()
```

The package initializes a default logger on load. Application startup reconfigures it via `SetGlobal()`, which also calls `slog.SetDefault()` so any code using the `slog` package directly shares the same configuration.

> [!IMPORTANT] Package-level convenience functions (`Debug`, `Info`, `Warn`, `Error`) were intentionally removed to enforce the injected logger pattern. All packages must obtain a `*logging.Logger` via struct injection or a package-level variable initialized with `WithComponent()`. This ensures `slog` records the actual caller's source file rather than `logger.go`.

## Imports / Dependencies

| Package | Purpose |
| ------- | ------- |
| `context` | Passed to `LogRequest` for context-aware logging |
| `fmt` | Error formatting, stderr fallback messages |
| `io` | `io.Writer`, `io.Closer`, `io.MultiWriter`, `io.Discard` |
| `log/slog` | Standard structured logging (handler creation, `SetDefault`) |
| `os` | `os.Stderr`, file operations for rotation |
| `path/filepath` | Directory and extension extraction for rotated filenames |
| `runtime` | `runtime.GOOS` and `runtime.GOARCH` for startup logging |
| `strings` | Extension trimming in rotation logic |
| `sync` | `sync.Mutex` for thread-safe file rotation |
| `time` | `time.Duration` for HTTP request duration logging |

## Side Effects

- **Package init:** `var global = Default()` creates a default console logger when the package is first imported.
- **SetGlobal:** Calls `slog.SetDefault()`, which changes the behavior of any code using the top-level `slog` package functions.
- **rotatingFileWriter:** On error, prints warnings to `os.Stderr` and disables file output for the remainder of the process lifetime.
- **New / NewFileLogger:** On file open failure, prints a warning to `os.Stderr` and falls back to console-only output (does not return an error in this case for `New`; `NewFileLogger` follows the same fallback pattern).
