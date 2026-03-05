---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/event-glossary-view-term/","title":"glossary:view-term","tags":["event-schema","glossary"]}
---


# `glossary:view-term`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend]]

> [!NOTE] Emitted when the user clicks a glossary mark in the ProseMirror editor, requesting the glossary panel to navigate to the clicked entry's detail view.

## Schema

```ts
{
  module: 'glossary';
  type: 'view-term';
  payload: {
    id: string;  // UUID of the glossary entry to view
  }
}
```

## Emitted By

| Source | Condition |
| ------ | --------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark#navigateToTerm\|TextEditor → GlossaryMarkProvider.navigateToTerm()]] | User clicks a `glossaryLink` mark in the ProseMirror editor (when click-to-navigate is enabled) |

## Consumed By

| Subscriber | Action Taken |
| ---------- | ------------ |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/GlossaryPanel#onMount\|GlossaryPanel → onMount subscription]] | Calls `glossaryStore.navigateToEntry()` which ensures the glossary panel is visible, fetches the entry by ID, and switches to the detail view |

## Payload Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | The UUID of the glossary entry that was clicked in the editor |

## Notes

> [!TIP] The `navigateToTerm` callback receives both `termId` and `termSlug` from the ProseMirror mark attributes, but only `id` is included in the event payload. The slug is unused because `navigateToEntry()` fetches by UUID.

The event is dispatched via `dispatchModuleEvent()` directly in the TextEditor component (not through a convenience function) because it originates from the ProseMirror plugin's callback wiring in TextEditor's `onMount`.
