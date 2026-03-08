---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/plugins/overview/","title":"Plugin System","tags":["feature","plugins","extensibility"],"updated":"2026-03-07T18:43:35.163-07:00"}
---


# Plugin System

> [!NOTE] Manifest-driven plugin architecture that enables third-party extensions to register custom content types, sidebar panels, context menu items, backend job handlers, and REST endpoints — with a permission-based security model, sandboxed APIs, and virtual content types that require no database migration.

## Overview

The Palimpsest plugin system allows developers to extend the application through a declarative manifest (`palimpsest-plugin.json`) that declares capabilities, permissions, and entry points. Plugins are discovered from the local file system (`DataDir/Plugins/`), validated against a JSON Schema, loaded via dynamic import, and integrated into the running application through registries.

The system is designed with a **two-tier architecture**:

| Tier | License | Capabilities | Backend Access |
| ---- | ------- | ------------ | -------------- |
| **Core** | Free | contentTypes, panels, contextMenuItems | Frontend only |
| **Pro** | Pro | All Core + jobHandlers, restEndpoints | Full backend access |

## Plugin Directory Structure

```
DataDir/Plugins/
└── my-plugin/
    ├── palimpsest-plugin.json   ← Plugin manifest
    ├── dist/
    │   └── index.js             ← Frontend JavaScript bundle
    └── assets/                  ← Static assets (icons, etc.)
```

## Plugin Manifest

Every plugin must have a `palimpsest-plugin.json` at its root. The manifest declares:

| Section | Purpose |
| ------- | ------- |
| Identity | `name`, `displayName`, `version`, `palimpsestVersion`, `description`, `author`, `license`, `repository` |
| Capabilities | Boolean flags for each extension point (`contentTypes`, `panels`, `contextMenuItems`, `jobHandlers`, `restEndpoints`) |
| Permissions | Resource access declarations (`fileSystem`, `network`, `localStorage`, `clipboard`) |
| Tier | `"core"` or `"pro"` — backend capabilities require `"pro"` |
| Entry Points | `frontend` (required), `backend` (optional, Pro only) |
| Definitions | Arrays of `contentTypes`, `panels`, `contextMenuItems`, `jobHandlers`, `restEndpoints` matching enabled capabilities |

### Manifest Validation

Both the Go backend (`[[modules/plugins-backend/discovery|discovery.go]]`) and Svelte frontend (`[[modules/plugins-frontend/manifestSchema|manifest-schema.ts]]`) validate manifests independently:

- **Required fields** — `name`, `displayName`, `version`, `palimpsestVersion`, `description`, `capabilities`, `permissions`, `tier`, `entryPoints`
- **Name format** — Lowercase, starts with a letter, only `[a-z0-9-]`, 2–64 characters
- **Version format** — Semantic versioning (`^\\d+\\.\\d+\\.\\d+`)
- **Tier constraints** — `jobHandlers` and `restEndpoints` capabilities require `tier: "pro"`
- **Capability–definition consistency** — If a capability is `true`, its corresponding definitions array must exist and be non-empty
- **Content type constraints** — `hasContent` must be `true`, `canHaveChildren` must be `false`, ID must match `^[a-z][a-z0-9-]*$`

## Extension Points

### Content Types (Core)

Plugins can register custom content types that appear alongside native types (chapter, scene, note, etc.) in the content tree. Each content type provides:

- A Svelte renderer component that replaces the default ProseMirror `TextEditor`
- A display name, icon, and optional color
- Parent constraints (`canBeChildOf: '*'` for any parent, or specific type IDs)
- Optional `onContentLoad` / `onContentSave` lifecycle hooks for data transformation
- Optional JSON Schema for content data validation

Content types are identified by a composite ID: `plugin:{pluginId}:{typeId}` (e.g., `plugin:excalidraw:drawing`). They are registered as **virtual content types** — frontend-only objects with `hierarchyName: 'plugin'`, `level: 999`, and `isVirtual: true` — requiring no database migration.

### Panels (Core)

Plugins can register sidebar or floating panel components:

- `defaultPosition`: `left`, `right`, `bottom`, or `floating`
- `allowedPositions`: Array of positions the user can move the panel to
- Panel component is exported from the frontend bundle

### Context Menu Items (Core)

Plugins can add items to the content tree context menu:

- Scoped to specific content types or `"*"` for all
- Handler function exported from the frontend bundle
- Optional keyboard shortcut
- Receives `ContextMenuContext` with content item, project slug, and plugin context

### Job Handlers (Pro Only)

Plugins can register backend job handlers for background processing tasks. Requires `tier: "pro"` and a backend entry point.

> [!WARNING] Job handlers are not yet implemented. The manifest schema accepts the definition, but the runtime integration with the backend job queue is a stub.

### REST Endpoints (Pro Only)

Plugins can register custom HTTP endpoints under `/api/plugins/{pluginId}/`. Requires `tier: "pro"` and a backend entry point.

> [!WARNING] REST endpoints are not yet implemented. The manifest schema accepts the definition, but the runtime routing is a stub.

## Permission Model

Plugins declare required permissions in their manifest. Each permission supports granular control:

| Permission | Values | Description |
| ---------- | ------ | ----------- |
| `fileSystem` | `false` \| `true` \| `string[]` | No access, full access, or restricted to specific paths |
| `network` | `false` \| `true` \| `string[]` | No access, unrestricted, or restricted to specific domains/origins |
| `localStorage` | `boolean` | Plugin-namespaced `localStorage` access |
| `clipboard` | `boolean` | Clipboard read/write access |

Permission checking is performed at runtime by `[[modules/plugins-frontend/security|security.ts]]`:

- **File system** — Path prefix matching (normalized to forward slashes, case-insensitive)
- **Network** — Origin matching with wildcard subdomain support (`*.example.com`)
- **Boolean permissions** — Simple flag check for `localStorage` and `clipboard`

All permission checks are audit-logged with timestamp, plugin ID, resource, and result. The log is bounded to 1000 entries (FIFO).

## Sandboxed APIs

Plugins receive sandboxed wrappers for system APIs via `[[modules/plugins-frontend/sandbox|sandbox.ts]]`:

| API | Sandbox Behavior |
| --- | ---------------- |
| `fetch` | Permission-checked against `network` declaration; 30-second default timeout via `AbortController`; blocked requests throw `PluginPermissionError` |
| `storage` | All keys namespaced with `plugin:{pluginId}:` prefix; returns `null` if `localStorage` permission not granted |
| `clipboard` | Returns `null` if `clipboard` permission not granted; wraps `navigator.clipboard` API |

## Plugin Lifecycle

### Discovery and Loading Flow

1. **Backend discovery** — `[[modules/plugins-backend/discovery|Discovery.DiscoverAll()]]` scans `DataDir/Plugins/` for directories containing `palimpsest-plugin.json`, parses and validates each manifest
2. **HTTP API** — Frontend fetches `GET /api/plugins` which returns all discovered plugins (valid and invalid, with error messages)
3. **Frontend validation** — `[[modules/plugins-frontend/manifestSchema|validateManifest()]]` re-validates the manifest on the client side
4. **Dynamic import** — Frontend module loaded via `import()` from `/api/plugins/{name}/assets/{entryPoint}`
5. **Content type registration** — Renderer components registered in `[[modules/plugins-frontend/contentRendererRegistry|ContentRendererRegistry]]`, virtual content types created
6. **Status transition** — Plugin moves through `loading` → `loaded` → `enabled` (or `error` at any step)

### Lifecycle Hooks

Plugins can implement hooks in a `lifecycle` export from their frontend module:

| Hook | When Called | Parameters |
| ---- | ---------- | ---------- |
| `onInstall` | First installation | `PluginContext` |
| `onUninstall` | Removal | `PluginContext` |
| `onEnable` | Plugin enabled | `PluginContext` |
| `onDisable` | Plugin disabled | `PluginContext` |
| `onProjectOpen` | Project opened | `PluginContext`, `projectSlug` |
| `onProjectClose` | Project closed | `PluginContext`, `projectSlug` |
| `onContentCreate` | New plugin content created | `PluginContext`, `parentContent` |
| `onContentLoad` | Plugin content loaded for rendering | `PluginContext`, `content` |
| `onContentSave` | Plugin content saved | `PluginContext`, `content`, `data` |
| `onContentDelete` | Plugin content deleted | `PluginContext`, `content` |

### Plugin API

Each plugin receives a `PluginAPI` object in its context providing:

| API | Methods | Implementation |
| --- | ------- | -------------- |
| `content` | `get(projectSlug, slugPath)`, `update(projectSlug, slugPath, data)` | HTTP fetch to `/api/projects/{slug}/content/{path}` |
| `notifications` | `success()`, `error()`, `info()`, `warning()` | Delegates to `notificationStore` |
| `events` | `on(module, type, callback)`, `dispatch(type, payload)` | Stub — planned integration with `ModuleEventBus` |
| `storage` | `get(key)`, `set(key, value)`, `remove(key)` | Namespaced `localStorage` with `plugin:{pluginId}:` prefix |

## Security

- **Dual-layer manifest validation** — Both Go backend and TypeScript frontend independently validate manifests against required fields, name/version formats, tier constraints, and capability–definition consistency
- **Directory traversal protection** — `ServePluginAsset` blocks paths containing `..` to prevent accessing files outside the plugin directory
- **Permission-gated API access** — All sandboxed APIs check permissions before executing; denied requests throw typed errors and are audit-logged
- **Tier enforcement** — Backend capabilities (`jobHandlers`, `restEndpoints`) are blocked for `core` tier plugins at the manifest validation level
- **Security audit warnings** — `[[modules/plugins-frontend/security|validatePluginSecurity()]]` warns on broad permissions (`fileSystem: true`, `network: true`), flags insecure HTTP URLs, and errors on Core tier plugins requesting Pro features
- **HTTPS preference** — Network permissions using plain HTTP (except localhost) generate security warnings
- **Namespaced isolation** — Plugin storage keys are prefixed with `plugin:{pluginId}:` to prevent cross-plugin data access

## Performance

- **Lazy component loading** — `ContentRendererRegistry.loadComponent()` dynamically imports renderer components on first use and caches them in a `Map` for subsequent renders
- **1-hour asset caching** — Backend sets `Cache-Control: public, max-age=3600` on served plugin assets to reduce repeated network requests
- **Non-blocking discovery** — Invalid plugins are included in discovery results with error messages rather than causing the entire discovery to fail; other plugins continue loading
- **30-second fetch timeout** — Sandboxed fetch enforces a timeout via `AbortController` to prevent plugin network calls from hanging indefinitely
- **Bounded audit log** — Permission request log is capped at 1000 entries with FIFO eviction to prevent unbounded memory growth

## Design Decisions

| Decision | Rationale |
| -------- | --------- |
| Manifest-driven architecture | Declarative manifests enable validation before code execution, static analysis, and marketplace integration |
| Virtual content types (no DB migration) | Plugins can add content types without schema changes; `isVirtual: true` flag distinguishes them in the type system |
| Dual-layer validation | Go backend validates for security (untrusted filesystem data); frontend validates for UX (immediate user feedback) |
| `plugin:{id}:{typeId}` ID format | Namespaced IDs prevent collisions between plugins and with native content types |
| Core/Pro tier separation | Free plugins are frontend-only (no server-side attack surface); backend access requires Pro license |
| Dynamic import for frontend modules | Avoids bundling plugin code into the main app; supports lazy loading and hot-reloading |
| Permission model with path/domain arrays | Granular permissions follow the principle of least privilege while remaining simple to declare |
| Sandboxed API wrappers | Permission checks happen at the API boundary, not inside plugin code — plugins cannot bypass them |

## Stubs and Future Work

> [!WARNING] The following features are defined in the type system and manifest schema but are not yet fully implemented at runtime:

| Feature | Status | Notes |
| ------- | ------ | ----- |
| Event bus integration | Stub | `PluginAPI.events.on()` / `dispatch()` return no-ops; planned integration with `ModuleEventBus` |
| Plugin settings from preferences | Stub | `PluginContext.settings` is always `{}` — planned to load from user preferences |
| Backend job handlers | Schema only | Manifest schema accepts definitions; no runtime job queue integration |
| Backend REST endpoints | Schema only | Manifest schema accepts definitions; no runtime routing integration |
| Plugin marketplace | Type only | `PluginSource: 'marketplace'` exists; no marketplace API or UI |
| `onInstall` / `onUninstall` hooks | Type only | Defined in `PluginLifecycle` but not called — no install/uninstall flow yet |

## Guides

- [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin\|Creating a Plugin: Step-by-Step Guide]] — Complete tutorial for building a plugin, using Excalidraw as the example

## Diagrams


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Plugin Discovery: Backend Scan to Frontend Load

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Content Rendering: Plugin Renderer Lifecycle

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Related

- [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend Module]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/overview\|Plugins Frontend Module]]
- [[Palimpsest-Writing-Software/api/plugins/overview\|Plugin API Service]]
- [[Palimpsest-Writing-Software/features/content-management/overview\|Content Management]]
- [[Palimpsest-Writing-Software/features/panel-system/overview\|Panel System]]
- [[Palimpsest-Writing-Software/features/editor/overview\|ProseMirror Editor]]
