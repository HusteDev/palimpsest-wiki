---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/api-client/","title":"client.ts","tags":["file","content-frontend","api"],"updated":"2026-03-05T06:23:55.436-07:00"}
---


# `client.ts`

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/api/client.ts`

> [!NOTE] Base HTTP API client for the Palimpsest backend. Provides typed HTTP methods (GET, POST, PUT, PATCH, DELETE) with automatic JSON serialization, structured error handling, timeout support, and logging. Used by all API modules.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `APIClientConfig` | interface | Client configuration (baseUrl, timeout) |
| `APIError` | class | Error with status, statusText, details, and convenience getters |
| `APIClient` | class | Main HTTP client class with typed methods |
| `apiClient` | const | Singleton APIClient instance |

## `APIClient` Class

### Constructor

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `config` | `APIClientConfig` | no | `{}` | Optional config overrides |
| `config.baseUrl` | `string` | no | `BACKEND_URL` | Backend base URL |
| `config.timeout` | `number` | no | `10000` | Request timeout in ms |

### Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `request<T>(endpoint, options?)` | `Promise<T>` | Main request method — all others delegate here |
| `get<T>(endpoint, options?)` | `Promise<T>` | GET request |
| `post<T>(endpoint, data?, options?)` | `Promise<T>` | POST request with JSON body |
| `put<T>(endpoint, data?, options?)` | `Promise<T>` | PUT request with JSON body |
| `patch<T>(endpoint, data?, options?)` | `Promise<T>` | PATCH request with JSON body |
| `delete<T>(endpoint, options?)` | `Promise<T>` | DELETE request |

### Request Behavior

- Sets `Content-Type: application/json` for POST/PUT requests
- Sets `X-Project-Type: local` header on all requests (Alpha: desktop only)
- Uses `AbortSignal.timeout()` for request timeout
- Handles 204 No Content by returning `{}`
- Parses JSON or text based on Content-Type header
- Throws `APIError` on non-2xx responses or network failures

## `APIError` Class

| Property | Type | Description |
| -------- | ---- | ----------- |
| `status` | `number` | HTTP status code (0 for network/timeout errors) |
| `statusText` | `string` | HTTP status text or error category |
| `details` | `string?` | Parsed error message from response body |
| `isAuthError` | `boolean` | True if 401 or 403 |
| `isNotFound` | `boolean` | True if 404 |
| `isNetworkError` | `boolean` | True if status is 0 |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `BACKEND_URL` | `$lib/config` | Default backend base URL |
| `createLogger` | `$lib/services/loggerService` | Logger ('api') |

## Side Effects

- Detects Tauri environment via `__TAURI__` or `__TAURI_INTERNALS__` on `window`
- Creates singleton `apiClient` at module load
