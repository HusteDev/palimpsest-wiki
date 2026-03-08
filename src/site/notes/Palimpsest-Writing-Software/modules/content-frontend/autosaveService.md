---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/autosave-service/","title":"autosaveService.ts","tags":["file","content-frontend","service","persistence"],"updated":"2026-03-05T06:24:55.432-07:00"}
---


# `autosaveService.ts`

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/services/autosaveService.ts`

> [!NOTE] Tier 3 of the three-tier persistence stack. Orchestrates automatic save triggers: time-based polling (configurable interval from user preferences) and navigation-based saves (three modes: enable, disable, prompt). Connects the localStorage draft layer to the backend sync layer.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `autosaveService` | singleton | AutosaveService instance |

## Key Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `register(projectSlug, contentId, slugPath)` | `void` | Register content for autosave monitoring |
| `unregister(projectSlug, contentId, performNavigationSave?)` | `Promise<void>` | Unregister with optional navigation save |
| `flushAll()` | `Promise<void>` | Force save all registered content immediately |
| `updateSettings()` | `void` | Rebuild timers from current user preferences |
| `isRegistered(projectSlug, contentId)` | `boolean` | Check if content is registered |
| `getRegisteredCount()` | `number` | Count of registered content items |
| `destroy()` | `void` | Cleanup all timers and state |

## Registration Flow

When `register()` is called (editor opens a content item):

1. Unregisters any existing registration for this content
2. Reads current user preferences for autosave settings
3. If `prefs.editor.autosave` is enabled and interval > 0, creates a `PollingTimer` with mode `polling`
4. Timer interval: `prefs.editor.autosaveInterval * 60 * 1000` (minutes to ms)
5. On timer fire: calls `performTimedSave()` → `saveService.saveContent()`
6. Starts monitoring user preferences for changes (rebuilds timers on change)

## Navigation Save Modes

When `unregister()` is called (navigating away from content):

| `saveOnNavigation` | Behavior |
| ------------------ | -------- |
| `enable` | Silently saves draft to backend via saveService |
| `disable` | Does nothing — draft stays in localStorage for recovery |
| `prompt` | Does nothing here — the editor page shows UnsavedChangesModal before calling unregister |

## Preference Monitoring

Subscribes to the `userPreferences` Svelte store. When preferences change, calls `updateSettings()` which destroys all existing timers and recreates them with the new interval.

## Page Unload

Registers a `beforeunload` handler that calls `flushAll()` to make a best-effort save of all registered content. Modern browsers may not guarantee async operations complete.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `localDraftService` | `./localDraftService` | Check for draft existence |
| `saveService` | `./saveService` | Backend sync |
| `userPreferences` | `$lib/stores/preferences` | Autosave settings |
| `get` | `svelte/store` | Read store value synchronously |
| `PollingTimer` | `./PollingTimer` | Polling timer for time-based saves |
| `createLogger` | `$lib/services/loggerService` | Logger ('autosave') |

## Side Effects

- Registers `beforeunload` event handler on `window` (module-level)
- Module-level singleton
- Subscribes to `userPreferences` store while content is registered
