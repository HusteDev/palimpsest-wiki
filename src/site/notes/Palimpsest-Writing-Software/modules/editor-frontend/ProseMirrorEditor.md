---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/prose-mirror-editor/","title":"ProseMirrorEditor","tags":["class","editor-frontend","prosemirror"],"updated":"2026-03-05T05:32:41.949-07:00"}
---


# `ProseMirrorEditor`

**File:** [[Palimpsest-Writing-Software/modules/editor-frontend/ProseMirrorEditor\|modules/editor-frontend/ProseMirrorEditor]]
**Extends:** None
**Implements:** None

> [!NOTE] Main editor wrapper class that encapsulates ProseMirror EditorView + EditorState creation, exposes a clean API for Svelte components, handles serialization (HTML/text/JSON), and installs all plugins in the correct order.

## Constructor

```ts
constructor(options: ProseMirrorEditorOptions)
```

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `options` | `ProseMirrorEditorOptions` | yes | Configuration including DOM element, initial content, callbacks |

### `ProseMirrorEditorOptions`

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `element` | `HTMLElement` | yes | — | DOM element to mount the editor into |
| `content` | `string` | no | `''` | Initial HTML content |
| `editable` | `boolean` | no | `true` | Whether the editor is editable |
| `placeholder` | `string` | no | `'Start writing...'` | Placeholder text for empty documents |
| `onUpdate` | callback | no | — | Called after document changes with `{ html, text, json, state }` |
| `onTransaction` | callback | no | — | Called on every transaction (including selection-only changes) |
| `onContextMenu` | `ContextMenuCallback` | no | — | Called on right-click with enriched context menu data |

## Properties

| Property | Type | Access | Description |
| -------- | ---- | ------ | ----------- |
| `view` | `EditorView` | public | The ProseMirror view instance |
| `state` | `EditorState` | public | Current editor state (updated on each transaction) |
| `options` | `ProseMirrorEditorOptions` | private | Constructor options (retained for editable/placeholder access) |
| `plugins` | `Plugin[]` | private | Assembled plugin list |

## Methods

### Serialization / Content Access

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `getHTML()` | `string` | Serialize current document to HTML via `DOMSerializer` |
| `getText()` | `string` | Get plain text content (concatenated text nodes) |
| `getJSON()` | `Record<string, unknown>` | Get ProseMirror JSON document representation |
| `setContent(content)` | `void` | Replace document from HTML; skips if content matches current HTML |
| `setContentFromJSON(json)` | `void` | Replace document from ProseMirror JSON; logs warning on parse failure |

### Editor State / View Control

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `setEditable(editable)` | `void` | Enable or disable editing |
| `focus()` | `void` | Focus the editor view |
| `blur()` | `void` | Blur the editor DOM |
| `chain()` | `CommandChain` | Start a fluent command chain |
| `can()` | `{ undo, redo }` | Capability checks for toolbar buttons |
| `isActive(name, attrs?)` | `boolean` | Check if a mark or node type is active at current selection |

### Search Highlights

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `setSearchHighlights(query)` | `void` | Set search query; dispatches meta to searchHighlight plugin |
| `nextSearchMatch()` | `void` | **STUB** — Navigate to next search match (future Find/Replace) |
| `previousSearchMatch()` | `void` | **STUB** — Navigate to previous match (future Find/Replace) |

### Text Analysis / Spellcheck

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `enableSpellcheck(provider)` | `void` | Set provider and dispatch 'enable' meta; schedules initial check |
| `disableSpellcheck()` | `void` | Clear provider and dispatch 'disable' meta |
| `getSpellcheckIssueAtPos(pos)` | `TextIssue \| null` | Get the spellcheck issue at a document position |
| `replaceWord(from, to, replacement)` | `void` | Replace a word range (for applying spelling suggestions) |
| `triggerSpellcheck()` | `void` | Schedule a re-check via the CheckManager |
| `getSpellingSuggestions(word)` | `string[]` | Get up to 5 suggestions from the active provider |

### Glossary Marks

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `setGlossaryMarkProvider(provider)` | `void` | Set the provider for tooltip and navigation support |
| `clearGlossaryMarkProvider()` | `void` | Clear the provider (call on editor unmount) |
| `clearGlossaryTooltipCache()` | `void` | Clear cached term data (call when glossary data changes) |

### Debug

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `getSelectionDebugInfo()` | object | Structured debug info for current selection context |
| `logSelectionDebugInfo()` | `void` | Log selection debug info to console |

### Lifecycle

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `destroy()` | `void` | Destroy the ProseMirror view and release all resources |

## Plugin Assembly

The `createPlugins()` method assembles plugins in this order (order matters):

1. `history()` — undo/redo stack
2. Custom keymap — `Mod-z` (undo), `Mod-y`/`Mod-Shift-z` (redo), mark shortcuts (`Mod-b`, `Mod-i`, `Mod-u`, `Mod-s`, `Mod-h`, `` Mod-` ``)
3. `baseKeymap` — default ProseMirror keybindings
4. `columnResizing()` — table column drag handles
5. `tableEditing()` — table cell selection
6. `dropCursor()` — visual drop cursor
7. `gapCursor()` — cursor for block boundaries
8. `fileHandler({ allowedMimeTypes })` — image drag-and-drop/paste
9. `createContextMenuPlugin(...)` — right-click handler (only if `onContextMenu` provided)
10. Placeholder plugin — widget decoration for empty documents
11. `createSearchHighlightPlugin()` — search match decorations
12. `createTextAnnotationPlugin()` — spellcheck decorations
13. `createGlossaryMarkPlugin()` — glossary tooltip/click

## Transaction Pipeline

The `dispatchTransaction` callback in the constructor handles the central transaction flow:

1. Apply transaction to state: `this.state = this.state.apply(transaction)`
2. Update view: `this.view.updateState(this.state)`
3. Fire `onTransaction` callback (always — for toolbar state updates, selection tracking)
4. If `transaction.docChanged`, fire `onUpdate` callback with serialized HTML, text, and JSON

> [!TIP] The `onTransaction` callback fires on selection-only changes too (not just content changes), making it suitable for updating toolbar active states and selection word counts in real time.

## Spellcheck-Aware Context Menu

The private `createSpellcheckAwareContextMenuCallback()` method wraps the user's `onContextMenu` callback to enrich `ContextMenuData` with spellcheck information:

1. Checks `getSpellcheckIssueAtPos(data.pos)` for a spelling issue at the click position
2. If found and type is `'spelling'`, populates `isMisspelled`, `misspelledWord`, `misspelledFrom`, `misspelledTo`, and `spellingSuggestions`
3. Forwards the enriched data to the user's callback

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `EditorState`, `Transaction`, `Plugin` | `prosemirror-state` | State management |
| `EditorView`, `Decoration`, `DecorationSet` | `prosemirror-view` | View and decorations |
| `history` | `prosemirror-history` | Undo/redo plugin |
| `keymap` | `prosemirror-keymap` | Keyboard shortcut plugin |
| `baseKeymap` | `prosemirror-commands` | Default keybindings |
| `dropCursor` | `prosemirror-dropcursor` | Drop cursor plugin |
| `gapCursor` | `prosemirror-gapcursor` | Gap cursor plugin |
| `columnResizing`, `tableEditing` | `prosemirror-tables` | Table plugins |
| `DOMParser`, `DOMSerializer` | `prosemirror-model` | Content parsing/serialization |
| `schema` | `./schema` | Document schema |
| Various commands | `./commands` | Mark toggles, undo/redo, CommandChain |
| `getSelectionDebugInfo` | `./pmUtils` | Debug helper |
| `fileHandler` | `./plugins/fileHandler` | Image file handling plugin |
| `createContextMenuPlugin`, types | `./plugins/contextMenu` | Context menu plugin |
| `createSearchHighlightPlugin`, key | `./plugins/searchHighlight` | Search decoration plugin |
| `createTextAnnotationPlugin`, key, helpers | `./plugins/textAnnotation` | Spellcheck decoration plugin |
| `createGlossaryMarkPlugin`, helpers, types | `./plugins/glossaryMark` | Glossary tooltip/click plugin |
| `TextAnalysisProvider`, `TextIssue` | `$lib/services/textAnalysis/types` | Spellcheck provider types |
| `createLogger` | `$lib/services/loggerService` | Logger (`'prosemirror'`) |

## Side Effects

- Creates a `ProseMirror EditorView` attached to the provided DOM element
- Installs DOM event listeners via plugins (mouse events, keyboard events, drag/drop)
- `destroy()` removes all listeners and DOM modifications
