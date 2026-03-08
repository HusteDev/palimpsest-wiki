---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/commands/","title":"commands.ts","tags":["file","editor-frontend","prosemirror"],"updated":"2026-03-05T05:32:41.353-07:00"}
---


# `commands.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/commands.ts`

> [!NOTE] 40+ ProseMirror commands for text formatting, block operations, structural transforms, and a fluent CommandChain utility with inter-command state refresh.

## Exports

### Types

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `Command` | type | `(state, dispatch?, view?) => boolean` — local command signature with optional view |

### Mark Commands (Inline Formatting)

| Name | Kind | Mark Type | Description |
| ---- | ---- | --------- | ----------- |
| `toggleBold` | const | `strong` | Toggle bold via `toggleMark` |
| `toggleItalic` | const | `em` | Toggle italic via `toggleMark` |
| `toggleCode` | const | `code` | Toggle inline code via `toggleMark` |
| `toggleStrikethrough` | const | `strikethrough` | Toggle strikethrough via `toggleMark` |
| `toggleUnderline` | const | `underline` | Toggle underline via `toggleMark` |
| `toggleHighlight` | const | `highlight` | Toggle highlight via `toggleMark` |

### Block Type Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `setParagraph` | const | Set block type to paragraph via `setBlockType` |
| `setHeading(level)` | function | Set block type to heading (1, 2, or 3) via `setBlockType` |

### List Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `toggleBulletList` | const | Toggle bullet list: lift if inside, swap if in ordered list, wrap if outside |
| `toggleOrderedList` | const | Toggle ordered list: lift if inside, swap if in bullet list, wrap if outside |
| `splitListItemCommand` | const | Split current list item at cursor |
| `liftListItemCommand` | const | Decrease list item indent |
| `sinkListItemCommand` | const | Increase list item indent |

### Wrapping Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `toggleBlockquote` | const | Toggle blockquote: lift if inside, wrap if outside |
| `setBlockquote` | const | Wrap selection in blockquote (non-toggle) |
| `wrapInBlockquoteManual` | const | Manual blockquote wrap using NodeRange + `findWrapping` |
| `toggleDetails` | const | Toggle details block: lift if inside, wrap with Slice preservation if outside |
| `wrapSelectionInDetailsBlockRange` | const | Details wrap using NodeRange (block-range only) |

### Insertion Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `insertHorizontalRule` | const | Insert `<hr>` at selection |
| `insertImage(src, alt?, title?)` | function | Insert inline image node |
| `insertTable(rows?, cols?, withHeaderRow?)` | function | Insert table (default 3x3 with header row) |

### Structural Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `duplicateSelection` | const | Duplicate current selection using Slice (preserves marks and structure) |

### Table Commands (Re-exported)

| Name | Description |
| ---- | ----------- |
| `addRowAfterCommand` / `addRowAfter` | Add row below current |
| `addColumnAfterCommand` / `addColumnAfter` | Add column right of current |
| `deleteRowCommand` / `deleteRow` | Delete current row |
| `deleteColumnCommand` / `deleteColumn` | Delete current column |
| `mergeCellsCommand` / `mergeCells` | Merge selected cells |
| `splitCellCommand` / `splitCell` | Split merged cell |
| `toggleHeaderRowCommand` / `toggleHeaderRow` | Toggle header row |
| `toggleHeaderColumnCommand` / `toggleHeaderColumn` | Toggle header column |

### History Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `undoCommand` | const | Undo via `prosemirror-history` |
| `redoCommand` | const | Redo via `prosemirror-history` |

### State Helpers

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `isMarkActive(state, markType)` | function | Check if a mark is active at current selection/cursor |
| `isNodeActive(state, nodeType, attrs?)` | function | Check if cursor is inside a node type (with optional attrs) |

### Debug Commands

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `logSelectionDebugInfo` | const | Log ResolvedPos paths + NodeRange to console |
| `toggleEntityTag(entityId)` | function | Example mark command with attrs (requires `entity_tag` mark in schema) |

### CommandChain Class

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `CommandChain` | class | Fluent command composition utility |

## Command Signature Adapter

All native ProseMirror commands (which take `(state, dispatch?)`) are wrapped via `fromPMCommand()` to match the local `Command` type `(state, dispatch?, view?)`. This provides a consistent interface across the codebase.

## List Toggle Logic

`toggleBulletList` and `toggleOrderedList` use three-way branching:

| Current Context | `toggleBulletList` Action | `toggleOrderedList` Action |
| --------------- | ------------------------ | ------------------------- |
| Inside bullet list | Lift (unwrap) | Swap to ordered list |
| Inside ordered list | Swap to bullet list | Lift (unwrap) |
| Outside any list | Wrap in bullet list | Wrap in ordered list |

List type detection uses `findNearestAncestorOfAnyType` from [[Palimpsest-Writing-Software/modules/editor-frontend/pmUtils\|pmUtils.ts]] and swaps the container via `setNodeMarkup`.

## Details Toggle Logic

`toggleDetails` handles three cases:

1. **Inside details:** Lift out (unwrap)
2. **Empty selection:** Create details with default "Details" summary + empty paragraph
3. **Non-empty selection:** Extract selection as Slice, wrap inline content in a paragraph (schema validation), create details with summary + selected content

## CommandChain

```ts
editor.chain().focus().toggleBold().run()
```

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `command(cmd)` | `this` | Add a raw command to the chain |
| `focus()` | `this` | Focus the editor view (not a PM command) |
| `run()` | `boolean` | Execute all commands sequentially |
| `toggleBold()` | `this` | Convenience: add `toggleBold` |
| `toggleItalic()` | `this` | Convenience: add `toggleItalic` |
| ... | `this` | (25+ convenience methods for all commands) |

> [!IMPORTANT] The `run()` method re-reads `view.state` before each command, ensuring later commands see changes from earlier ones. This is critical for commands that depend on `storedMarks`, resolved positions, node ranges, or structure after a previous transformation.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `EditorState`, `Transaction`, `TextSelection`, `NodeSelection` | `prosemirror-state` | State and selection types |
| `EditorView` | `prosemirror-view` | View for dispatch |
| `NodeType` | `prosemirror-model` | Node type checking |
| `findWrapping` | `prosemirror-transform` | Structural wrapping |
| `toggleMark`, `setBlockType`, `wrapIn`, `lift` | `prosemirror-commands` | Core PM commands |
| `wrapInList`, `splitListItem`, `liftListItem`, `sinkListItem` | `prosemirror-schema-list` | List commands |
| `undo`, `redo` | `prosemirror-history` | History commands |
| 8 table commands | `prosemirror-tables` | Table operations |
| `schema` | `./schema` | Schema reference for node/mark types |
| Various utilities | `./pmUtils` | Ancestor lookup, block range, slice helpers |
| `createLogger` | `$lib/services/loggerService` | Logger (`'pm-commands'`) |

## Side Effects

None — all functions are pure commands that only dispatch transactions when called with a `dispatch` function.
