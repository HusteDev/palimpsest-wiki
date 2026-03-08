---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-backend/retention/","title":"retention.go","tags":["file","logging-backend"],"updated":"2026-03-05T07:13:19.412-07:00"}
---


# `retention.go`

**Module:** [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend]]
**Path:** `backend/internal/logging/retention.go`

> [!NOTE] Log file retention and cleanup utilities for removing old log files at startup.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `CleanupOldLogs(dir string, retentionDays int)` | func | Deletes `.log` files older than `retentionDays` in the given directory |

## CleanupOldLogs

```go
func CleanupOldLogs(dir string, retentionDays int) (deleted int, err error)
```

**Parameters:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| `dir` | `string` | Directory to scan for `.log` files |
| `retentionDays` | `int` | Number of days to retain log files. `0` means keep forever (no-op). |

**Returns:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| `deleted` | `int` | Number of files successfully deleted |
| `err` | `error` | Error if the directory cannot be read; individual file delete errors are not returned |

**Behavior:**

- `retentionDays == 0` (or negative): returns immediately with `(0, nil)`. All files are kept.
- `retentionDays > 0`: calculates a cutoff time (`now - N days`), then iterates over directory entries.
- Only files with the `.log` extension are considered. Subdirectories are skipped.
- File age is determined by modification time (`info.ModTime()`).
- Files older than the cutoff are deleted via `os.Remove`.
- Individual delete errors and stat errors are logged to `os.Stderr` with `[logging] WARNING:` prefix but do not stop processing of remaining files and are not included in the returned error.
- Returns an error only if `os.ReadDir` fails to read the directory itself.

**Invocation context:** Called once at startup after the logger completes Phase 2 reconfiguration (i.e., after `SetGlobal` is called with the file-backed logger). This ensures the logger is available to record the cleanup result.

## Imports / Dependencies

| Package | Purpose |
| ------- | ------- |
| `fmt` | Error formatting and stderr warning output |
| `os` | `os.ReadDir`, `os.Remove`, `os.Stderr` |
| `path/filepath` | `filepath.Join` to build full paths for deletion |
| `strings` | `strings.HasSuffix` and `strings.ToLower` for `.log` extension check |
| `time` | `time.Now().AddDate` for cutoff calculation |

## Side Effects

- Deletes files from the filesystem matching the retention policy.
- Prints warning messages to `os.Stderr` when individual files cannot be stat'd or deleted.
