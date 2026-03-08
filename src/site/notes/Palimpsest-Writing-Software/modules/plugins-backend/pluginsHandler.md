---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-backend/plugins-handler/","title":"plugins.go (Handler)","tags":["file","plugins-backend"],"updated":"2026-03-05T07:56:19.404-07:00"}
---


# `plugins.go` (Handler)

**Module:** [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend]]
**Path:** `backend/internal/http/handlers/plugins.go`

> [!NOTE] HTTP handler for plugin management endpoints. Provides plugin discovery listing, individual plugin lookup, and static asset serving for plugin frontend bundles with directory traversal protection and MIME-based content type detection.

## Exports

### Struct

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `PluginHandler` | struct | Handles all plugin-related HTTP requests. Holds `discovery` (*plugins.Discovery) and `log` (*logging.Logger, component `"handler-plugins"`). |

### Functions / Methods

| Name | Kind | Parameters | Returns | Description |
| ---- | ---- | ---------- | ------- | ----------- |
| `NewPluginHandler` | constructor | `discovery *plugins.Discovery` -- the plugin discovery service | `*PluginHandler` | Creates a new PluginHandler with an injected logger using the `"handler-plugins"` component name. |
| `Plugins` | method on `*PluginHandler` | `w http.ResponseWriter`; `r *http.Request` | _(none)_ | Main route dispatcher for `/api/plugins` and all sub-paths. Parses the URL path and delegates to the appropriate sub-handler. |
| `ListPlugins` | method on `*PluginHandler` | `w http.ResponseWriter`; `r *http.Request` | _(none)_ | Handles `GET /api/plugins`. Calls `discovery.DiscoverAll()` and returns a `PluginListResponse` JSON payload containing all discovered plugins (valid and invalid). Returns `500` on fatal discovery errors. |
| `GetPlugin` | method on `*PluginHandler` | `w http.ResponseWriter`; `r *http.Request`; `pluginName string` -- the plugin identifier extracted from the URL path | _(none)_ | Handles `GET /api/plugins/{name}`. Checks plugin existence, then discovers all plugins and filters for the requested one. Returns the single `DiscoveredPlugin` JSON payload, or `404` if not found. |
| `ServePluginAsset` | method on `*PluginHandler` | `w http.ResponseWriter`; `r *http.Request` | _(none)_ | Handles `GET /api/plugins/{name}/assets/*`. Parses plugin name and asset path from the URL, applies directory traversal protection (rejects paths containing `".."`), verifies the plugin exists, then delegates to `serveFile`. Returns `400` for invalid paths, `404` for missing plugins or assets. |
| `serveFile` | method on `*PluginHandler` (unexported) | `w http.ResponseWriter`; `r *http.Request`; `filePath string` -- absolute path to the file to serve | _(none)_ | Serves a static file with automatic MIME type detection via `mime.TypeByExtension`. Falls back to `"application/javascript"` for `.js` files and `"application/octet-stream"` for unknown extensions. Sets `Cache-Control: public, max-age=3600` (1-hour cache). Returns `404` if the file does not exist or is a directory. |

## Route Dispatch

The `Plugins()` method parses the URL path and routes to internal handlers:

| URL Pattern | Method | Handler |
| ----------- | ------ | ------- |
| `/api/plugins` | GET | `ListPlugins` |
| `/api/plugins/{name}/assets/*` | GET | `ServePluginAsset` |
| `/api/plugins/{name}` | GET | `GetPlugin` |

Non-GET requests to list and single-plugin endpoints return `405 Method Not Allowed`. Unmatched paths return `404 Not Found`.

## Security: Directory Traversal Protection

`ServePluginAsset` checks for `".."` in the asset path before constructing the full file path. Any request containing `".."` in the asset portion is rejected with a `400 Bad Request` and a warning-level log entry. This prevents attackers from escaping the plugin directory to read arbitrary files on the server.

## Response Caching

`serveFile` sets the following cache header on all served assets:

```
Cache-Control: public, max-age=3600
```

This instructs browsers and intermediate proxies to cache plugin assets for 1 hour.

## MIME Type Detection

`serveFile` determines the `Content-Type` header using the following priority:

1. `mime.TypeByExtension(ext)` -- standard library lookup based on file extension
2. `.js` files fall back to `"application/javascript"` if the standard lookup returns empty
3. All other unknown extensions fall back to `"application/octet-stream"`

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `io` | stdlib | `io.Copy` for streaming file content to the response writer |
| `mime` | stdlib | `TypeByExtension` for MIME type detection |
| `net/http` | stdlib | HTTP handler types, `http.Dir` for file serving, status codes |
| `path/filepath` | stdlib | `Ext` for extracting file extension |
| `strings` | stdlib | URL path parsing (`TrimPrefix`, `SplitN`, `Contains`, `Split`) |
| `response` | `backend/internal/http/response` | Standardized HTTP response helpers (`OK`, `BadRequest`, `NotFound`, `MethodNotAllowed`, `InternalError`) |
| `logging` | `backend/internal/logging` | Structured logging via injected `*logging.Logger` with component `"handler-plugins"` |
| `plugins` | `backend/internal/plugins` | `Discovery` service and `PluginListResponse` type |

## Side Effects

- **HTTP response writing:** All handler methods write HTTP responses (status codes, headers, JSON bodies) to the `http.ResponseWriter`. These are standard HTTP side effects.
- **File system reads:** `ServePluginAsset` and `serveFile` open and read files from the plugin directory via `http.Dir("/").Open(filePath)`.
- **Logging:** All methods emit structured log messages at DEBUG, WARN, and ERROR levels through the injected logger.

## Notes

> [!WARNING] `GetPlugin` calls `discovery.DiscoverAll()` and then filters the result, rather than discovering a single plugin directly. The source code includes a comment acknowledging this could be optimized. For projects with many plugins, this may become a performance concern.

> [!WARNING] The directory traversal check in `ServePluginAsset` uses a simple `strings.Contains(assetPath, "..")` check. While sufficient for the current use case, this does not use `filepath.Clean` or other canonicalization. The check is applied before path joining, which prevents the most common traversal attacks.

> [!WARNING] `serveFile` uses `http.Dir("/").Open(filePath)` to open files. This roots the file server at the filesystem root, relying on the earlier traversal check and `GetPluginAssetPath` to restrict access to the correct directory. Ensure the traversal protection in `ServePluginAsset` is maintained if this code is modified.
