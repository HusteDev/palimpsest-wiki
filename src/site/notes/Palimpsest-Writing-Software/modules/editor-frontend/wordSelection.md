---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/word-selection/","title":"wordSelection.ts","tags":["file","editor-frontend","prosemirror"]}
---


# `wordSelection.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/wordSelection.ts`

> [!NOTE] Word boundary detection and selection utilities for ProseMirror. Used by the context menu for smart word auto-selection on right-click and by the toolbar for selection-based word/character counts.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `WordAtPos` | interface | Word boundary result type |
| `findWordAtPos` | function | Find the word under a given document position |
| `selectWordAtPos` | function | Select the word at a position in the editor |
| `getSelectedText` | function | Get the currently selected text from the editor |

## `WordAtPos` Interface

| Field | Type | Description |
| ----- | ---- | ----------- |
| `from` | `number` | Start position of the word in document coordinates |
| `to` | `number` | End position of the word in document coordinates |
| `word` | `string` | The word text |

## Key Functions

### `findWordAtPos(doc, pos)`

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `doc` | `PMNode` | yes | The ProseMirror document node |
| `pos` | `number` | yes | Document position to find the word at |

**Returns:** `WordAtPos | null`

Resolves the position, extracts the parent textblock's text content (replacing non-text inline nodes with `\uFFFC` placeholder), then uses a Unicode-aware regex pattern (`/[\w\u00C0-\u024F\u1E00-\u1EFF]+/g`) to find word boundaries. Returns `null` if the position is not inside a textblock or not on a word.

> [!TIP] The regex covers Latin extended characters (accented chars like e, n, u) and Latin Extended Additional, making it suitable for most European languages. CJK and other scripts are not currently handled.

### `selectWordAtPos(view, pos)`

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `view` | `EditorView` | yes | The ProseMirror editor view |
| `pos` | `number` | yes | Position to select the word at |

**Returns:** `boolean` — `true` if a word was selected

Calls `findWordAtPos`, then dispatches a `TextSelection` spanning the word boundaries. Used by the [[Palimpsest-Writing-Software/modules/editor-frontend/contextMenu\|context menu plugin]] to auto-select the word under the right-click cursor when no text is already selected.

### `getSelectedText(view)`

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `view` | `EditorView` | yes | The ProseMirror editor view |

**Returns:** `string` — selected text, or empty string if no selection

Uses `doc.textBetween(from, to, ' ')` to extract text with space separators between blocks. Called by [[Palimpsest-Writing-Software/modules/editor-frontend/TextEditor\|TextEditor.svelte]] on every transaction for selection word/character count reporting.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Node` | `prosemirror-model` | Document node type |
| `TextSelection` | `prosemirror-state` | Selection creation |
| `EditorView` | `prosemirror-view` | View access for dispatch |

## Side Effects

None.
