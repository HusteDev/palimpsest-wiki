---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/context-menu/","title":"contextMenu.ts","tags":["file","editor-frontend","prosemirror","plugin"]}
---


# `contextMenu.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/plugins/contextMenu.ts`

> [!NOTE] ProseMirror plugin that intercepts right-click events to show a custom context menu. Performs smart word auto-selection when no text is selected and detects the editing context (table, list, blockquote, details, code block).

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `EditorContext` | interface | Context flags for the click location |
| `ContextMenuData` | interface | Full data payload passed to the context menu callback |
| `ContextMenuCallback` | type | `(data: ContextMenuData) => void` |
| `createContextMenuPlugin` | function | Factory that creates the context menu plugin |

## `EditorContext` Interface

| Field | Type | Description |
| ----- | ---- | ----------- |
| `inTable` | `boolean` | Inside a `table` node |
| `inList` | `boolean` | Inside a `bullet_list` or `ordered_list` |
| `inBlockquote` | `boolean` | Inside a `blockquote` |
| `inDetails` | `boolean` | Inside a `details` block |
| `inCodeBlock` | `boolean` | Inside a `code_block` (if schema has one) |

## `ContextMenuData` Interface

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `x` | `number` | — | Screen X coordinate for menu positioning |
| `y` | `number` | — | Screen Y coordinate for menu positioning |
| `pos` | `number` | — | Editor document position at click |
| `selectedText` | `string` | — | Selected text (after potential word auto-selection) |
| `hadExistingSelection` | `boolean` | — | Whether there was already a selection before the click |
| `context` | `EditorContext` | — | Context flags for the click location |
| `isMisspelled` | `boolean` | `false` | Whether the word at click has a spelling error |
| `misspelledWord` | `string` | `''` | The misspelled word text |
| `misspelledFrom` | `number` | `0` | Document start of misspelled word |
| `misspelledTo` | `number` | `0` | Document end of misspelled word |
| `spellingSuggestions` | `string[]` | `[]` | Up to 5 spelling suggestions |

> [!TIP] The spellcheck fields (`isMisspelled`, `misspelledWord`, `spellingSuggestions`) default to empty values in this plugin. They are populated by [[Palimpsest-Writing-Software/modules/editor-frontend/ProseMirrorEditor\|ProseMirrorEditor.ts]]'s `createSpellcheckAwareContextMenuCallback()` which has access to the textAnnotation plugin state.

## Behavior

### Event Handling

The plugin registers a `contextmenu` DOM event handler via `handleDOMEvents`:

1. Prevents the browser's default context menu
2. Converts mouse coordinates to a ProseMirror document position via `view.posAtCoords()`
3. If no text is selected, calls `selectWordAtPos()` from [[Palimpsest-Writing-Software/modules/editor-frontend/wordSelection\|wordSelection.ts]] to auto-select the word under the cursor
4. Gets the (potentially updated) selected text
5. Detects editing context via `detectContext()` using `isInsideNodeType()` from [[Palimpsest-Writing-Software/modules/editor-frontend/pmUtils\|pmUtils.ts]]
6. Invokes the callback with the assembled `ContextMenuData`
7. Returns `true` to indicate the event was handled

### Context Detection

`detectContext()` checks each context flag by calling `isInsideNodeType()` with the appropriate schema node type. The checks use ResolvedPos ancestor traversal (depth walk) from the current selection.

## Decision Points

| Condition | Branch |
| --------- | ------ |
| `posAtCoords()` returns null | Return true (click outside editor content, no menu) |
| Selection is empty | Auto-select word at click position |
| Selection already exists | Use existing selection text |
| `schema.nodes.code_block` exists | Check `inCodeBlock` context |
| `schema.nodes.code_block` missing | `inCodeBlock = false` |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Plugin` | `prosemirror-state` | Plugin factory |
| `EditorView` | `prosemirror-view` | View for coordinate/position conversion |
| `schema` | `../schema` | Node type references |
| `selectWordAtPos`, `getSelectedText` | `../wordSelection` | Word selection utilities |
| `isInsideNodeType`, `findNearestAncestorOfAnyType` | `../pmUtils` | Context detection |

## Side Effects

- Prevents the browser's default context menu (`event.preventDefault()`)
- May dispatch a selection transaction (via `selectWordAtPos`) to auto-select a word
