---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-backend/sanitize/","title":"sanitize.go","tags":["file","logging-backend"],"updated":"2026-03-05T07:14:00.414-07:00"}
---


# `sanitize.go`

**Module:** [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend]]
**Path:** `backend/internal/logging/sanitize.go`

> [!NOTE] Sensitive field redaction utilities that prevent accidental logging of passwords, tokens, API keys, and other secrets.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SanitizeValue(fieldName, value string)` | func | Returns `"[REDACTED]"` if the field name matches a sensitive pattern, else the original value |
| `SanitizeHeaders(headers http.Header)` | func | Returns a sanitized map of HTTP headers with sensitive values redacted |
| `SanitizeQueryParams(query url.Values)` | func | Returns a sanitized map of URL query parameters with sensitive values redacted |

## Constants

| Name | Value | Description |
| ---- | ----- | ----------- |
| `redactedPlaceholder` | `"[REDACTED]"` | Replacement string for any sensitive value |

## Sensitive Patterns

The package maintains an internal list of 11 case-insensitive substring patterns. Any field name containing one of these substrings triggers redaction:

| # | Pattern |
| - | ------- |
| 1 | `password` |
| 2 | `passwd` |
| 3 | `token` |
| 4 | `secret` |
| 5 | `key` |
| 6 | `authorization` |
| 7 | `cookie` |
| 8 | `api_key` |
| 9 | `apikey` |
| 10 | `credential` |
| 11 | `session` |

Matching is performed by `isSensitiveField` (unexported), which lowercases the field name and checks for substring containment against each pattern.

## Function Details

### isSensitiveField (unexported)

```go
func isSensitiveField(fieldName string) bool
```

**Parameters:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| `fieldName` | `string` | The field, header, or parameter name to check |

**Returns:** `true` if the lowercased field name contains any of the 11 sensitive patterns.

### SanitizeValue

```go
func SanitizeValue(fieldName, value string) string
```

**Parameters:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| `fieldName` | `string` | Name of the field (e.g., `"password"`, `"Authorization"`) |
| `value` | `string` | The field's value |

**Returns:** `"[REDACTED]"` if the field name is sensitive, otherwise the original `value` unchanged.

**Example:**

```go
logging.SanitizeValue("password", "my-secret")    // "[REDACTED]"
logging.SanitizeValue("username", "alice")         // "alice"
logging.SanitizeValue("X-Api-Key", "abc123")       // "[REDACTED]"
```

### SanitizeHeaders

```go
func SanitizeHeaders(headers http.Header) map[string]string
```

**Parameters:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| `headers` | `http.Header` | HTTP headers to sanitize |

**Returns:** A `map[string]string` where each header name maps to either `"[REDACTED]"` (if sensitive) or the header values joined with `", "` (for multi-value headers). Safe to pass directly to a structured logger.

### SanitizeQueryParams

```go
func SanitizeQueryParams(query url.Values) map[string]string
```

**Parameters:**

| Name | Type | Description |
| ---- | ---- | ----------- |
| `query` | `url.Values` | URL query parameters to sanitize |

**Returns:** A `map[string]string` where each parameter name maps to either `"[REDACTED]"` (if sensitive) or the parameter values joined with `", "` (for multi-value parameters). Safe to pass directly to a structured logger.

## Imports / Dependencies

| Package | Purpose |
| ------- | ------- |
| `net/http` | `http.Header` type for `SanitizeHeaders` |
| `net/url` | `url.Values` type for `SanitizeQueryParams` |
| `strings` | `strings.ToLower` for case-insensitive matching, `strings.Contains` for substring check, `strings.Join` for multi-value concatenation |

## Side Effects

None. All functions are pure with no global state mutation, I/O, or package-level initialization beyond the `sensitivePatterns` slice which is immutable after init.
