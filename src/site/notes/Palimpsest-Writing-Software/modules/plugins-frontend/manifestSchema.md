---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/plugins-frontend/manifest-schema/","title":"manifest-schema.ts","tags":["file","plugins-frontend"],"updated":"2026-03-05T07:57:17.401-07:00"}
---


# `manifest-schema.ts`

**Module:** `[[modules/plugins-frontend/overview|Plugins Frontend]]`
**Path:** `src/lib/plugins/manifest-schema.ts`

> [!NOTE] JSON Schema definition and lightweight validation for `palimpsest-plugin.json` manifest files. Used by `[[modules/plugins-frontend/pluginLoader|PluginLoader]]` to ensure manifests are well-formed before loading.

## Exports

| Name | Kind | Description |
| --- | --- | --- |
| `pluginManifestSchema` | const (JSONSchemaType) | JSON Schema Draft 7 definition for the plugin manifest. Defines required fields (`name`, `displayName`, `version`, `palimpsestVersion`, `description`, `capabilities`, `permissions`, `tier`, `entryPoints`), property constraints, and cross-field validation rules via `allOf` conditions |
| `validateManifest` | function | Validates a manifest object against the schema using lightweight checks (no Ajv dependency). Accepts `unknown` input. Returns a `ManifestValidationResult` |
| `ManifestValidationResult` | interface | Validation result with `valid` boolean and `errors` array of `ManifestValidationError` |
| `ManifestValidationError` | interface | A single validation error with `path` (JSON path like `"/contentTypes/0/id"`), `message`, and optional `value` |

## Imports / Dependencies

| Import | Source | Purpose |
| --- | --- | --- |
| (none) | -- | This file has no external dependencies. It is self-contained with only a local `JSONSchemaType` type alias |

## Validation Rules

The `validateManifest` function performs the following checks without requiring the Ajv library:

**Required fields:** Ensures all 9 required top-level fields are present.

**Name format:** Must match regex `^[a-z][a-z0-9-]*$` (lowercase, starts with letter, letters/numbers/hyphens only). Length must be 2-64 characters.

**Version format:** Must match semver regex `^\d+\.\d+\.\d+(-[a-zA-Z0-9.]+)?(\+[a-zA-Z0-9.]+)?$`.

**Tier values:** Must be `"core"` or `"pro"`.

**Capabilities:** All 5 capability fields (`contentTypes`, `panels`, `contextMenuItems`, `jobHandlers`, `restEndpoints`) must be booleans. Backend capabilities (`jobHandlers`, `restEndpoints`) require `tier: "pro"`.

**Permissions:** `localStorage` and `clipboard` must be booleans. `fileSystem` must be a boolean or string array. `network` must be a boolean or string array.

**Entry points:** `frontend` must be a string.

**Content types (conditional):** When `capabilities.contentTypes` is `true`, the `contentTypes` array must exist and be non-empty. Each entry is validated for: `id` format (same regex as plugin name), `renderer` presence, `hasContent` must be `true`, `canHaveChildren` must be `false`.

## Schema Cross-Field Rules

The `pluginManifestSchema.allOf` array defines conditional rules intended for full JSON Schema validators:

| Condition | Requirement |
| --- | --- |
| `capabilities.contentTypes === true` | `contentTypes` array must exist with at least 1 item |
| `capabilities.panels === true` | `panels` array must exist with at least 1 item |
| `capabilities.contextMenuItems === true` | `contextMenuItems` array must exist with at least 1 item |
| `capabilities.jobHandlers === true` OR `capabilities.restEndpoints === true` | `tier` must be `"pro"` |
| `entryPoints.backend` is present | At least one of `capabilities.jobHandlers` or `capabilities.restEndpoints` must be `true` |

## Side Effects

None. This file exports a static schema constant and a pure validation function.

## Notes

> [!WARNING] The `validateManifest` function is a lightweight reimplementation that does not use Ajv. It covers the most critical validation checks but does not enforce every constraint in the full JSON Schema (e.g., `additionalProperties: false` is not checked by the function). The full schema object is exported separately for use with proper JSON Schema validators if needed.

> [!WARNING] The `JSONSchemaType` alias defined in this file is a simplified type, not a complete JSON Schema Draft 7 type definition. It exists only to provide basic typing for the schema constant.
