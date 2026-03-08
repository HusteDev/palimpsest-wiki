---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/sandbox/","title":"sandbox.ts","tags":["file","plugins-frontend"],"updated":"2026-03-05T07:58:20.408-07:00"}
---


# `sandbox.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/sandbox.ts`

> [!NOTE] Sandboxed API wrappers for plugins. Provides permission-checked, namespaced versions of system APIs (fetch, localStorage, clipboard) that enforce the permissions declared in each plugin's manifest.

## Exports

| Name | Kind | Description |
| --- | --- | --- |
| `SandboxedFetchOptions` | interface | Extends `RequestInit` with an optional `timeout` property (milliseconds, default 30000) |
| `createSandboxedFetch` | function | Factory that returns a permission-checked fetch function for a plugin. Checks network permission via `[[modules/plugins-frontend/security\|checkPermission]]` before each request, logs the permission check via `logPermissionRequest`, enforces a timeout using `AbortController`, and throws `PluginPermissionError` on denial or `PluginTimeoutError` on timeout |
| `SandboxedStorage` | interface | Namespaced localStorage wrapper with methods: `getItem`, `setItem`, `removeItem`, `clear`, `keys`, and a `length` getter |
| `createSandboxedStorage` | function | Factory that creates a `SandboxedStorage` instance for a plugin, or returns `null` if localStorage permission is denied. All keys are automatically prefixed with `plugin:{pluginId}:`. The `clear` method only removes keys with the plugin's prefix. The `keys` method returns unprefixed key names |
| `SandboxedClipboard` | interface | Permission-checked clipboard interface with `readText()` and `writeText(text)` methods |
| `createSandboxedClipboard` | function | Factory that creates a `SandboxedClipboard` instance for a plugin, or returns `null` if clipboard permission is denied. Wraps `navigator.clipboard` methods with error handling |
| `SandboxedAPI` | interface | Complete sandboxed API bundle: `fetch` (sandboxed fetch function), `storage` (`SandboxedStorage \| null`), `clipboard` (`SandboxedClipboard \| null`), `pluginId` |
| `createSandboxedAPI` | function | Main factory that assembles a complete `SandboxedAPI` for a plugin. Calls `createSandboxedFetch`, `createSandboxedStorage`, and `createSandboxedClipboard` internally |
| `PluginSandboxError` | class | Base error class for all sandbox errors. Extends `Error` with `name: 'PluginSandboxError'` |
| `PluginPermissionError` | class | Thrown when a plugin attempts to access a resource without permission. Extends `PluginSandboxError` with `name: 'PluginPermissionError'` |
| `PluginTimeoutError` | class | Thrown when a sandboxed fetch request exceeds the timeout. Extends `PluginSandboxError` with `name: 'PluginTimeoutError'` |
| `PluginStorageError` | class | Thrown when a sandboxed storage `setItem` operation fails (e.g., quota exceeded). Extends `PluginSandboxError` with `name: 'PluginStorageError'` |
| `PluginClipboardError` | class | Thrown when a sandboxed clipboard operation fails. Extends `PluginSandboxError` with `name: 'PluginClipboardError'` |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| `createLogger` | `$lib/services/loggerService` | Structured logging with `plugin-sandbox` component name |
| `PluginPermissions` | `[[modules/plugins-frontend/pluginTypes\|./types]]` | Permission declarations from plugin manifest |
| `checkPermission` | `[[modules/plugins-frontend/security\|./security]]` | Permission checking for each sandboxed operation |
| `logPermissionRequest` | `[[modules/plugins-frontend/security\|./security]]` | Audit logging of permission checks |

## Error Class Hierarchy

```
Error
  └── PluginSandboxError (base)
        ├── PluginPermissionError
        ├── PluginTimeoutError
        ├── PluginStorageError
        └── PluginClipboardError
```

## Side Effects

- **Network requests:** `createSandboxedFetch` returns a function that performs `fetch()` calls with `AbortController` timeout enforcement.
- **localStorage access:** `createSandboxedStorage` reads and writes to `localStorage` using the `plugin:{pluginId}:` prefix namespace.
- **Clipboard access:** `createSandboxedClipboard` calls `navigator.clipboard.readText()` and `navigator.clipboard.writeText()`.
- **Audit logging:** Every factory function logs a permission check via `logPermissionRequest` from `[[modules/plugins-frontend/security|security.ts]]`. The sandboxed fetch function logs each individual request's permission check.

## Notes

> [!WARNING] The default fetch timeout is 30 seconds (30000ms). This can be overridden per-request via `SandboxedFetchOptions.timeout`. Timed-out requests are aborted via `AbortController.abort()`.

> [!WARNING] `createSandboxedStorage` and `createSandboxedClipboard` return `null` (not an error-throwing wrapper) when the plugin lacks the corresponding permission. Consumers must null-check before use.

> [!WARNING] The `SandboxedStorage.clear()` method iterates all `localStorage` keys to find those matching the plugin prefix. This is a linear scan and could be slow if `localStorage` contains a very large number of keys.

> [!WARNING] Clipboard operations may fail due to browser security policies (e.g., requires user gesture, secure context). These failures are wrapped in `PluginClipboardError`.
