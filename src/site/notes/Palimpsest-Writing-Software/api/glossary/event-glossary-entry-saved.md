---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/event-glossary-entry-saved/","title":"glossary:entry-saved","tags":["event-schema","glossary"]}
---


# `glossary:entry-saved`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend]]

> [!NOTE] Emitted after a glossary entry is successfully created or updated, notifying the editor and other components that glossary data has changed and content marks may need refreshing.

## Schema

```ts
{
  module: 'glossary';
  type: 'entry-saved';
  payload: {
    id: string;  // UUID of the glossary entry that was saved
  }
}
```

## Emitted By

| Source | Condition |
| ------ | --------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/GlossaryEntryForm#handleSave\|GlossaryEntryForm → handleSave()]] | After a successful `glossaryStore.createEntry()` call (create mode) |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/GlossaryEntryForm#handleSave\|GlossaryEntryForm → handleSave()]] | After a successful `glossaryStore.updateEntry()` call (edit mode) |

Dispatched via the convenience function `dispatchGlossaryEntrySaved(entryId)` from [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|moduleEventBus]].

## Consumed By

| Subscriber | Action Taken |
| ---------- | ------------ |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|TextEditor → onMount subscription]] | Calls `editor.clearGlossaryTooltipCache()` to invalidate cached tooltip data so the next hover fetches fresh definitions |
| Editor page (`+page.svelte`) | After a 3-second delay, reloads the currently selected content document to pick up any new `glossaryLink` marks applied by the background TermScanHandler |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/GlossaryPanel\|GlossaryEntryDetail → onMount subscription]] | After a 4-second delay, reloads the entry's usage/occurrence count to reflect marks applied by the background scan |

## Payload Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | The UUID of the glossary entry that was created or updated |

## Notes

> [!WARNING] The 3-second and 4-second delays in subscribers are heuristics to allow the backend TermScanHandler background job to complete before reloading. If the scan takes longer than expected, the editor may briefly show stale mark state. A subsequent manual content reload will pick up the final marks.

> [!TIP] This event triggers the spellcheck dictionary integration indirectly: `GlossaryEntryForm` calls `dictionaryStore.addWordIfMisspelled()` for the term and all aliases after dispatching this event (with a 2-second delay).
