---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/local-draft-service/","title":"localDraftService.ts","tags":["file","content-frontend","service","persistence"],"updated":"2026-03-05T06:24:16.436-07:00"}
---


# `localDraftService.ts`

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/services/localDraftService.ts`

> [!NOTE] Tier 1 of the three-tier persistence stack. Manages debounced auto-save of editor content to browser localStorage with 3-second delay. Provides draft recovery on editor mount and status events for UI feedback. Data source: browser localStorage (ephemeral client-side cache).

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `DraftData` | interface | Draft content structure (html, text, json, wordCount, charCount, lastModified) |
| `DraftStatusEvent` | type | Discriminated union of status events |
| `DraftStatusCallback` | type | Callback for status events |
| `localDraftService` | singleton | LocalDraftService instance |

## `DraftData` Interface

| Field | Type | Description |
| ----- | ---- | ----------- |
| `html` | `string` | Serialized HTML content |
| `text` | `string` | Plain text content |
| `json` | `Record<string, unknown>` | ProseMirror JSON document |
| `wordCount` | `number` | Word count at time of draft |
| `charCount` | `number` | Character count at time of draft |
| `lastModified` | `string` | ISO timestamp of last modification |

## Status Events

| Type | Description |
| ---- | ----------- |
| `draft_saving` | Debounce timer started/reset (save pending) |
| `draft_saved` | Draft successfully written to localStorage |
| `draft_recovered` | Draft recovered from localStorage on mount |
| `draft_error` | Failed to read/write localStorage |

## Key Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `scheduleSave(projectSlug, contentId, data)` | `void` | Start debounced save timer (resets on each call) |
| `getDraft(projectSlug, contentId)` | `DraftData \| null` | Read draft from localStorage |
| `hasDraft(projectSlug, contentId)` | `boolean` | Check if draft exists |
| `isDraftNewer(projectSlug, contentId, backendTimestamp)` | `boolean` | Compare draft timestamp with backend |
| `clearDraft(projectSlug, contentId)` | `void` | Remove draft from localStorage |
| `flushPendingSave(projectSlug, contentId)` | `Promise<void>` | Flush pending save immediately |
| `flushAll()` | `Promise<void>` | Flush all pending saves (page unload) |
| `unregister(projectSlug, contentId)` | `void` | Cleanup timer and pending data |
| `notifyRecovered(projectSlug, contentId)` | `void` | Emit draft_recovered event for UI |
| `onStatusChange(callback)` | `() => void` | Register status callback, returns unsubscribe |
| `destroy()` | `void` | Cleanup all timers and state |

## Storage Keys

- Draft data: `{LOCAL_STORAGE_PREFIX}{projectSlug}-{contentId}`
- Content tracking: `{projectSlug}/{contentId}` (internal key for timers/events)

## Timer Behavior

Uses `PollingTimer` with `mode: 'restart'` (debounce behavior). Each call to `scheduleSave()` resets the timer to `DRAFT_SAVE_DELAY` (3 seconds). The timer fires once and stops (`onExpire` returns `false`).

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `DRAFT_SAVE_DELAY` | `$lib/config` | Debounce delay (3000ms) |
| `LOCAL_STORAGE_PREFIX` | `$lib/config` | localStorage key prefix |
| `PollingTimer` | `./PollingTimer` | Timer with debounce mode |
| `createLogger` | `$lib/services/loggerService` | Logger ('draft') |

## Side Effects

- Reads/writes browser localStorage
- Module-level singleton persists across component lifecycles
