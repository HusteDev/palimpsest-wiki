---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/plugins/overview/","title":"Plugin API Service","tags":["api","plugins"],"updated":"2026-03-05T07:55:59.408-07:00"}
---


# Plugin API Service

> [!NOTE] REST API service for plugin discovery, retrieval, and static asset serving. Endpoints scan `DataDir/Plugins/` for installed plugins and expose their manifests and bundled frontend assets.

## Base URL

`/api/plugins`

## Authentication

None (Alpha: local desktop app only).

---

## Endpoints

### `GET /api/plugins`

List all discovered plugins. The backend scans the `DataDir/Plugins/` directory for subdirectories containing a valid `palimpsest-plugin.json` manifest. Plugins with invalid or missing manifests are still included in the response, but carry a populated `error` field.

**Handler:** [[Palimpsest-Writing-Software/modules/plugins-backend/pluginsHandler#ListPlugins\|PluginHandler.ListPlugins]]

#### Request

No path parameters, query parameters, or request body.

#### Response -- `200 OK`

```json
{
  "plugins": [
    {
      "name": "screenplay-editor",
      "path": "/home/user/.palimpsest/Plugins/screenplay-editor",
      "source": "local",
      "manifest": {
        "name": "screenplay-editor",
        "displayName": "Screenplay Editor",
        "version": "1.0.0",
        "palimpsestVersion": ">=0.1.0",
        "description": "Industry-standard screenplay formatting.",
        "capabilities": {
          "contentTypes": true,
          "panels": false,
          "contextMenuItems": false,
          "jobHandlers": false,
          "restEndpoints": false
        },
        "permissions": {
          "fileSystem": false,
          "network": false,
          "localStorage": true,
          "clipboard": false
        },
        "tier": "core",
        "entryPoints": {
          "frontend": "dist/index.js"
        },
        "contentTypes": []
      }
    },
    {
      "name": "broken-plugin",
      "path": "/home/user/.palimpsest/Plugins/broken-plugin",
      "source": "local",
      "manifest": {},
      "error": "invalid manifest JSON: unexpected end of JSON input"
    }
  ]
}
```

**Schema:** `PluginListResponse` wrapping an array of `DiscoveredPlugin` objects.

#### Error Responses

| Status | Condition | Body |
| ------ | --------- | ---- |
| `405 Method Not Allowed` | Non-GET request | — |
| `500 Internal Server Error` | Cannot read plugins directory | `{ error: "failed to discover plugins" }` |

---

### `GET /api/plugins/{pluginName}`

Retrieve a single discovered plugin by its manifest `name`. Returns the same `DiscoveredPlugin` shape as the list endpoint but for one plugin only.

**Handler:** [[Palimpsest-Writing-Software/modules/plugins-backend/pluginsHandler#GetPlugin\|PluginHandler.GetPlugin]]

#### Request

**Path Parameters:**

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `pluginName` | string | Plugin identifier (lowercase, alphanumeric + hyphens). Must match a directory name under `DataDir/Plugins/`. |

No query parameters or request body.

#### Response -- `200 OK`

```json
{
  "name": "screenplay-editor",
  "path": "/home/user/.palimpsest/Plugins/screenplay-editor",
  "source": "local",
  "manifest": {
    "name": "screenplay-editor",
    "displayName": "Screenplay Editor",
    "version": "1.0.0",
    "palimpsestVersion": ">=0.1.0",
    "description": "Industry-standard screenplay formatting.",
    "capabilities": { "contentTypes": true, "panels": false, "contextMenuItems": false, "jobHandlers": false, "restEndpoints": false },
    "permissions": { "fileSystem": false, "network": false, "localStorage": true, "clipboard": false },
    "tier": "core",
    "entryPoints": { "frontend": "dist/index.js" },
    "contentTypes": []
  }
}
```

#### Error Responses

| Status | Condition | Body |
| ------ | --------- | ---- |
| `404 Not Found` | Plugin directory does not exist or name not matched | `{ error: "plugin not found" }` |
| `405 Method Not Allowed` | Non-GET request | — |
| `500 Internal Server Error` | Discovery failed after existence check passed | `{ error: "failed to discover plugins" }` |

---

### `GET /api/plugins/{pluginName}/assets/{assetPath...}`

Serve a static file from a plugin's directory. This is the primary mechanism for loading plugin frontend bundles (JavaScript, CSS) and any other assets (images, fonts) a plugin ships.

**Handler:** [[Palimpsest-Writing-Software/modules/plugins-backend/pluginsHandler#ServePluginAsset\|PluginHandler.ServePluginAsset]]

#### Request

**Path Parameters:**

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `pluginName` | string | Plugin identifier. |
| `assetPath` | string | Relative path within the plugin directory. Supports nested paths (e.g., `dist/index.js`, `assets/icon.png`). |

No query parameters or request body.

#### Response -- `200 OK`

The raw file contents with appropriate headers:

| Header | Value | Notes |
| ------ | ----- | ----- |
| `Content-Type` | Auto-detected | Determined by `mime.TypeByExtension` from Go stdlib. Falls back to `application/javascript` for `.js` files, or `application/octet-stream` for unknown extensions. |
| `Cache-Control` | `public, max-age=3600` | 1-hour browser cache. |

#### Error Responses

| Status | Condition | Body |
| ------ | --------- | ---- |
| `400 Bad Request` | Asset path contains `..` (traversal attempt) or is malformed | `{ error: "invalid asset path" }` |
| `404 Not Found` | Plugin does not exist, file not found, or path resolves to a directory | `{ error: "plugin not found" }` or `{ error: "asset not found" }` |
| `405 Method Not Allowed` | Non-GET request | — |
| `500 Internal Server Error` | File stat failed after open | `{ error: "failed to read asset" }` |

---

## Security

> [!NOTE] Palimpsest Core is a local desktop application with no network-facing server. All plugin API endpoints are unauthenticated.

### Directory Traversal Protection

The asset-serving endpoint explicitly rejects any `assetPath` that contains `..`. If a traversal attempt is detected, the handler logs a warning and returns `400 Bad Request` without touching the filesystem beyond the initial string check.

### No Authentication

All three endpoints are served without authentication or authorization. This is appropriate for the Core license tier where the backend runs as a Tauri sidecar accessible only from the local desktop application.

---

## Asset Serving Notes

### MIME Type Detection

The handler uses Go's `mime.TypeByExtension` to map file extensions to MIME types from the system MIME database. Two special cases apply:

- `.js` files always resolve to `application/javascript`, even if the system database returns empty.
- Unknown extensions fall back to `application/octet-stream` (binary download).

### Caching Behavior

All successfully served assets receive a `Cache-Control: public, max-age=3600` header, allowing browsers and Tauri's webview to cache plugin assets for one hour. There is no cache-busting mechanism at the API level; plugins that need cache invalidation should include version hashes in their bundle filenames.

---

## DiscoveredPlugin Schema

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `name` | string | yes | Plugin identifier from the manifest (or directory name if manifest is invalid). |
| `path` | string | yes | Absolute filesystem path to the plugin directory. |
| `source` | string | yes | Where the plugin was found. Currently always `"local"`. Future values: `"npm"`, `"marketplace"`. |
| `manifest` | `PluginManifest` | yes | Parsed manifest object. Empty object `{}` when the manifest could not be parsed. |
| `error` | string | no | Present only when discovery encountered a problem (missing manifest, invalid JSON, validation failure). |

For full manifest schema details, see [[Palimpsest-Writing-Software/features/plugins/overview\|features/plugins/overview]].

---

## Handler Implementation

- Plugin routes: [[Palimpsest-Writing-Software/modules/plugins-backend/pluginsHandler\|plugins.go]]
- Discovery service: [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugin Discovery]]

## Related

- [[Palimpsest-Writing-Software/features/plugins/overview\|Plugins Feature]]
- [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend Module]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/overview\|Plugins Frontend Module]]
