---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/security/","title":"security.ts","tags":["file","plugins-frontend"],"updated":"2026-03-07T13:47:00.336-07:00"}
---


# `security.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/security.ts`

> [!NOTE] Plugin security module for permission checking, validation, and audit logging. Ensures plugins only access resources they have declared in their manifest permissions.

## Exports

| Name | Kind | Description |
| --- | --- | --- |
| `PermissionCheckResult` | interface | Result of a permission check: `allowed` boolean and optional `reason` string explaining denial |
| `PermissionType` | type | Union literal `'fileSystem' \| 'network' \| 'localStorage' \| 'clipboard'` |
| `checkPermission` | function | Main permission dispatcher. Accepts `PluginPermissions`, a `PermissionType`, and an optional `resource` string. Dispatches to type-specific internal checkers and returns a `PermissionCheckResult` |
| `summarizePermissions` | function | Generates an array of `PermissionSummary` objects for UI display. Produces one summary per permission type with human-readable labels, descriptions, and access levels |
| `PermissionSummary` | interface | Human-readable permission summary: `type` (PermissionType), `label`, `description`, `level` (`'none' \| 'restricted' \| 'full'`), optional `details` string array |
| `validatePluginSecurity` | function | Validates a `PluginManifest` for security concerns. Returns a `SecurityValidationResult` with `valid` boolean, `warnings` array, and `errors` array |
| `SecurityValidationResult` | interface | Security validation result: `valid` boolean, `warnings` string array, `errors` string array |
| `logPermissionRequest` | function | Records a permission request to the in-memory audit log. Accepts `pluginId`, `type`, optional `resource`, and `allowed` boolean. Log is bounded to `MAX_LOG_SIZE` (1000 entries) using FIFO eviction |
| `getPermissionLog` | function | Retrieves recent permission requests. Optionally filters by `pluginId`. Accepts a `limit` parameter (default 100). Returns the most recent entries |
| `clearPermissionLog` | function | Empties the in-memory permission audit log |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| `createLogger` | `$lib/services/loggerService` | Structured logging with `plugin-security` component name |
| `PluginPermissions` | `[[modules/plugins-frontend/pluginTypes\|./types]]` | Permission declarations from plugin manifest |
| `PluginManifest` | `[[modules/plugins-frontend/pluginTypes\|./types]]` | Full manifest for security validation |

## Internal Functions

| Name | Kind | Description |
| --- | --- | --- |
| `checkFileSystemPermission` | function | Handles file system permission checks. `false` denies all access. `true` grants full access. Array of paths uses prefix matching against the normalized requested path |
| `checkNetworkPermission` | function | Handles network permission checks. `false` denies all access. `true` grants full access. Array of origins uses exact origin matching and wildcard subdomain matching (e.g., `*.example.com` matches any subdomain of `example.com`) |
| `checkBooleanPermission` | function | Handles simple boolean permissions (`localStorage`, `clipboard`). Returns allowed if `true`, denied with descriptive reason if `false` |
| `normalizePath` | function | Converts backslashes to forward slashes and lowercases the path for cross-platform comparison |
| `summarizeFileSystemPermission` | function | Generates a `PermissionSummary` for file system access with appropriate level and description |
| `summarizeNetworkPermission` | function | Generates a `PermissionSummary` for network access with appropriate level and description |

## Security Validation Checks

The `validatePluginSecurity` function inspects a manifest for the following concerns:

| Check | Severity | Condition |
| --- | --- | --- |
| Full file system access | warning | `permissions.fileSystem === true` |
| Unrestricted network access | warning | `permissions.network === true` |
| Core plugin using job handlers | error | `tier === 'core'` and `capabilities.jobHandlers === true` |
| Core plugin using REST endpoints | error | `tier === 'core'` and `capabilities.restEndpoints === true` |
| Insecure HTTP network URLs | warning | Network permission array contains non-HTTPS URLs (excluding `localhost` and `127.0.0.1`) |

## Side Effects

- **Module-level state:** An in-memory `permissionLog` array (bounded to 1000 entries) accumulates permission request records for the lifetime of the application. Entries are added by `logPermissionRequest` and evicted FIFO when the limit is reached.
- **Logging:** All permission checks and audit log entries produce debug-level structured log output via `createLogger`.

## Notes

> [!WARNING] The permission audit log is in-memory only and is lost on page refresh. It is intended for runtime debugging and settings UI display, not persistent auditing.

> [!WARNING] File system path matching uses simple prefix comparison after normalization (backslash-to-forward-slash, lowercase). This is not glob-based and does not handle symlinks or relative path traversal.

> [!WARNING] Network wildcard subdomain matching (`*.example.com`) matches any hostname ending with `.example.com`. It does not validate the scheme (HTTP vs HTTPS) separately from the origin check.
