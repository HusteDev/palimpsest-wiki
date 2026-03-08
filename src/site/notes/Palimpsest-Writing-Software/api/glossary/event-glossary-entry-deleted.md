---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/event-glossary-entry-deleted/","title":"glossary:entry-deleted","tags":["event-schema","glossary"],"updated":"2026-03-05T05:32:37.152-07:00"}
---


# `glossary:entry-deleted`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend]]

> [!NOTE] Emitted after a glossary entry is successfully deleted, notifying the editor and other components that marks should be refreshed and cached tooltip data invalidated.

## Schema

```ts
{
  module: 'glossary';
  type: 'entry-deleted';
  payload: {
    id: string;  // UUID of the glossary entry that was deleted
  }
}
```

## Emitted By

| Source | Condition |
| ------ | --------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store#deleteEntry\|glossaryStore.deleteEntry()]] | After a successful `glossaryApi.deleteEntry()` call completes and the entry is removed from the local store state |

Dispatched via the convenience function `dispatchGlossaryEntryDeleted(entryId)` from [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|moduleEventBus]].

## Consumed By

| Subscriber | Action Taken |
| ---------- | ------------ |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|TextEditor → onMount subscription]] | Calls `editor.clearGlossaryTooltipCache()` to invalidate cached tooltip data for the deleted entry |
| Editor page (`+page.svelte`) | After a 3-second delay, reloads the currently selected content document to pick up mark removals applied by the backend "remove" background job |

## Payload Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | The UUID of the glossary entry that was deleted |

## Notes

> [!WARNING] The 3-second delay in the editor page subscriber is a heuristic to allow the backend `TermScanHandler.executeRemove()` background job to finish stripping `glossaryLink` marks from all content documents. If the removal takes longer, marks for the deleted entry may briefly remain visible until the next content reload.

> [!IMPORTANT] Unlike `glossary:entry-saved`, this event is dispatched from the `glossaryStore` (not from `GlossaryEntryForm`) because deletion is triggered from the `GlossaryEntryDetail` component via the store's `deleteEntry()` method.
