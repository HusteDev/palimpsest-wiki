---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/logging-backend/errors/","title":"errors.go","tags":["file","logging-backend"],"updated":"2026-03-05T07:13:45.417-07:00"}
---


# `errors.go`

**Module:** [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend]]
**Path:** `backend/internal/logging/errors.go`

> [!NOTE] Application error types, sentinel errors, typed constructors, and utility functions for consistent error handling across the backend.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `ErrNotFound` | sentinel error | "not found" |
| `ErrAlreadyExists` | sentinel error | "already exists" |
| `ErrInvalidInput` | sentinel error | "invalid input" |
| `ErrUnauthorized` | sentinel error | "unauthorized" |
| `ErrForbidden` | sentinel error | "forbidden" |
| `ErrInternal` | sentinel error | "internal error" |
| `ErrDatabaseError` | sentinel error | "database error" |
| `ErrValidation` | sentinel error | "validation error" |
| `AppError` | struct | Rich application error with user message, error code, HTTP status, and detail map |
| `NewAppError(err, message, httpStatus)` | func | Generic `*AppError` constructor |
| `NotFoundError(resource)` | func | Creates a 404 `*AppError` wrapping `ErrNotFound` |
| `AlreadyExistsError(resource)` | func | Creates a 409 `*AppError` wrapping `ErrAlreadyExists` |
| `ValidationError(message)` | func | Creates a 400 `*AppError` wrapping `ErrValidation` |
| `InvalidInputError(message)` | func | Creates a 400 `*AppError` wrapping `ErrInvalidInput` |
| `UnauthorizedError(message)` | func | Creates a 401 `*AppError` wrapping `ErrUnauthorized` |
| `ForbiddenError(message)` | func | Creates a 403 `*AppError` wrapping `ErrForbidden` |
| `InternalError(err)` | func | Creates a 500 `*AppError` wrapping the given error directly |
| `DatabaseError(err)` | func | Creates a 500 `*AppError` wrapping `ErrDatabaseError` with the original error |
| `WrapError(err, message)` | func | Wraps an error with `fmt.Errorf` using `%w` |
| `IsNotFound(err)` | func | Checks `errors.Is(err, ErrNotFound)` |
| `IsAlreadyExists(err)` | func | Checks `errors.Is(err, ErrAlreadyExists)` |
| `IsValidation(err)` | func | Checks both `ErrValidation` and `ErrInvalidInput` |
| `GetHTTPStatus(err)` | func | Extracts HTTP status via `errors.As`; defaults to 500 |
| `GetErrorCode(err)` | func | Extracts app error code via `errors.As`; defaults to `"INTERNAL_ERROR"` |
| `GetUserMessage(err)` | func | Extracts user-friendly message via `errors.As`; defaults to `"an error occurred"` |

## AppError Struct

```go
type AppError struct {
    Err        error            // Underlying error (for unwrapping / sentinel checks)
    Message    string           // User-friendly error message (safe to return to clients)
    Code       string           // Application-specific error code (e.g., "NOT_FOUND")
    HTTPStatus int              // Suggested HTTP response status code
    Details    map[string]any   // Additional error context (lazily initialized)
}
```

**Methods:**

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `Error()` | `string` | Returns `Message` if set, else `Err.Error()`, else `"unknown error"` |
| `Unwrap()` | `error` | Returns `Err` for `errors.Is` / `errors.As` chain support |
| `WithDetails(key string, value any)` | `*AppError` | Adds a key-value pair to `Details` (initializes map if nil); returns self for chaining |

## Typed Constructors

Each constructor creates an `*AppError` pre-configured with the appropriate sentinel error, error code, and HTTP status.

| Constructor | Sentinel | Code | HTTP Status | Message Pattern |
| ----------- | -------- | ---- | ----------- | --------------- |
| `NotFoundError(resource)` | `ErrNotFound` | `NOT_FOUND` | 404 | `"{resource} not found"` |
| `AlreadyExistsError(resource)` | `ErrAlreadyExists` | `ALREADY_EXISTS` | 409 | `"{resource} already exists"` |
| `ValidationError(message)` | `ErrValidation` | `VALIDATION_ERROR` | 400 | caller-provided message |
| `InvalidInputError(message)` | `ErrInvalidInput` | `INVALID_INPUT` | 400 | caller-provided message |
| `UnauthorizedError(message)` | `ErrUnauthorized` | `UNAUTHORIZED` | 401 | caller-provided or `"authentication required"` |
| `ForbiddenError(message)` | `ErrForbidden` | `FORBIDDEN` | 403 | caller-provided or `"access denied"` |
| `InternalError(err)` | wraps `err` directly | `INTERNAL_ERROR` | 500 | `"an internal error occurred"` |
| `DatabaseError(err)` | `ErrDatabaseError` (wraps original) | `DATABASE_ERROR` | 500 | `"a database error occurred"` |

`UnauthorizedError` and `ForbiddenError` provide default messages when the caller passes an empty string. `InternalError` wraps the caller-provided error directly (not a sentinel) so the original error is preserved for logging but the user-facing message is generic. `DatabaseError` double-wraps: the `Err` field is `fmt.Errorf("%w: %v", ErrDatabaseError, err)`, preserving both the sentinel and the original error details.

## Utility Functions

### WrapError

```go
func WrapError(err error, message string) error
```

Returns `nil` if `err` is `nil`. Otherwise returns `fmt.Errorf("%s: %w", message, err)`, preserving the error chain for `errors.Is` / `errors.As`.

### IsNotFound / IsAlreadyExists / IsValidation

Type-checking helpers that use `errors.Is` to walk the error chain. `IsValidation` checks for both `ErrValidation` and `ErrInvalidInput` since both represent client-side input problems.

### GetHTTPStatus / GetErrorCode / GetUserMessage

Extraction helpers that use `errors.As` to unwrap to `*AppError`. If the error is not an `AppError`, they return safe defaults:

| Function | Default |
| -------- | ------- |
| `GetHTTPStatus` | `500` (`http.StatusInternalServerError`) |
| `GetErrorCode` | `"INTERNAL_ERROR"` |
| `GetUserMessage` | `"an error occurred"` |

## Imports / Dependencies

| Package | Purpose |
| ------- | ------- |
| `errors` | `errors.New` for sentinels, `errors.Is` / `errors.As` for type checks |
| `fmt` | `fmt.Sprintf` for message formatting, `fmt.Errorf` with `%w` for wrapping |
| `net/http` | HTTP status constants (`http.StatusNotFound`, etc.) |

## Side Effects

None. This file defines types, constructors, and pure utility functions with no package-level init or global state mutation.
