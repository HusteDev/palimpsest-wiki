---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/save-service/","title":"saveService.ts","tags":["file","content-frontend","service","persistence"],"updated":"2026-03-05T06:24:33.431-07:00"}
---


# `saveService.ts`

**Module:** [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
**Path:** `src/lib/services/saveService.ts`

> [!NOTE] Tier 2 of the three-tier persistence stack. Synchronizes content from localStorage to the backend database. First flushes any pending localStorage save, then transfers the draft data to the backend API, clears the draft on success, and tracks unsynced documents for crash recovery.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `SaveContentOptions` | interface | Options (immediate: skip debouncing) |
| `SaveStatusEvent` | type | Discriminated union of save status events |
| `SaveStatusCallback` | type | Callback for save status events |
| `saveService` | singleton | SaveService instance |

## Status Events

| Type | Description |
| ---- | ----------- |
| `backend_saving` | Backend sync started |
| `backend_saved` | Backend sync completed successfully |
| `backend_error` | Backend sync failed |

## Key Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `saveContent(projectSlug, contentId, slugPath, options?)` | `Promise<{ success, error? }>` | Full save pipeline: flush draft → read → sync → clear |
| `hasUnsyncedChanges(projectSlug, contentId)` | `boolean` | Check if document has unsynced changes |
| `getUnsyncedDocuments()` | `string[]` | Get all unsynced document keys |
| `loadUnsyncedMarkers()` | `void` | Restore unsynced state from localStorage on startup |
| `onStatusChange(callback)` | `() => void` | Register status callback |
| `destroy()` | `void` | Cleanup resources |

## Save Pipeline

The `saveContent()` method executes this pipeline:

1. **Flush localStorage** — Calls `localDraftService.flushPendingSave()` to ensure latest data is persisted
2. **Read draft** — Calls `localDraftService.getDraft()` to get the draft data
3. **Mark unsynced** — Stores unsynced marker in localStorage for crash recovery
4. **Recalculate word count** — Uses `wordCountService.countAllFromJson()` for accuracy
5. **API call** — Calls `contentApi.updateContent()` with content JSON and word count
6. **On success**: Updates reactive word count store, clears draft, clears unsynced marker
7. **On failure**: Emits `backend_error` event, returns `{ success: false, error }`

## Unsynced Tracking

Documents that have been saved to localStorage but not yet to the backend are tracked:
- **In-memory**: `unsyncedDocuments` Set
- **In localStorage**: `unsynced-{projectSlug}/{contentId}` keys with ISO timestamps
- **Recovery**: `loadUnsyncedMarkers()` scans localStorage on startup to restore the Set

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `contentApi` | `$lib/api/content` | Backend API calls |
| `localDraftService` | `./localDraftService` | Flush and read drafts |
| `wordCountService` | `./wordCountService` | Word count recalculation and store update |
| `createLogger` | `$lib/services/loggerService` | Logger ('save') |

## Side Effects

- Reads/writes browser localStorage (unsynced markers)
- Module-level singleton
