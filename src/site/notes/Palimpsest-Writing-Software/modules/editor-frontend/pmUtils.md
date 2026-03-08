---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/pm-utils/","title":"pmUtils.ts","tags":["file","editor-frontend","prosemirror"],"updated":"2026-03-05T05:32:41.840-07:00"}
---


# `pmUtils.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/pmUtils.ts`

> [!NOTE] Centralizes ProseMirror tree/selection utilities — ResolvedPos ancestor traversal, NodeRange creation, and Slice extraction — so command code stays readable.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `AncestorMatch` | interface | Return type for ancestor lookup functions |
| `findNearestAncestorOfType` | function | Find closest ancestor matching a single NodeType |
| `findNearestAncestorOfAnyType` | function | Find closest ancestor matching any of multiple NodeTypes |
| `isInsideNodeType` | function | Boolean check: is selection inside a given node type? |
| `getBlockRange` | function | Build a block-level NodeRange for current selection |
| `getSelectionSlice` | function | Extract a structure-preserving Slice from selection |
| `sliceContentToNodeArray` | function | Convert a Slice's content fragment to a flat node array |
| `getSelectionDebugInfo` | function | Return structured debug object for selection context |
| `findAncestorByPredicate` | function | Generalized ancestor search with custom predicate |

## `AncestorMatch` Interface

| Field | Type | Description |
| ----- | ---- | ----------- |
| `node` | `PMNode` | The matched ancestor node |
| `depth` | `number` | Depth in the ResolvedPos path |
| `pos` | `number` | Absolute position of the node (usable in `setNodeMarkup`, `replaceWith`, etc.) |
| `start` | `number` | Absolute start position of the node's content |
| `end` | `number` | Absolute end position of the node's content |

## Key Functions

### `findNearestAncestorOfType(state, nodeType)`

Walks the ResolvedPos path from `$from.depth` upward to depth 1. Uses `$from.node(depth)` to check each ancestor against the target `nodeType`. Returns `AncestorMatch` or `null`.

**Called by:** [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] (list toggle, blockquote toggle), [[Palimpsest-Writing-Software/modules/editor-frontend/contextMenu\|contextMenu.ts]] (context detection)

### `findNearestAncestorOfAnyType(state, nodeTypes)`

Same traversal as above but matches against an array of node types. Used for "am I inside bullet_list OR ordered_list?" checks.

**Called by:** [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] (`getContainingListInfo`)

### `isInsideNodeType(state, nodeType)`

Convenience boolean wrapper around `findNearestAncestorOfType`. Returns `true` if any ancestor matches.

**Called by:** [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] (`toggleBlockquote`, `toggleDetails`), [[Palimpsest-Writing-Software/modules/editor-frontend/contextMenu\|contextMenu.ts]] (`detectContext`)

### `getBlockRange(state)`

Delegates to `$from.blockRange($to)` to get a ProseMirror `NodeRange`. Returns `null` if ProseMirror cannot build a valid block range for the current selection.

**Called by:** [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] (`wrapInBlockquoteManual`, `wrapSelectionInDetailsBlockRange`, `getSelectionDebugInfo`)

### `getSelectionSlice(state)`

Extracts a `Slice` from `state.doc.slice(from, to)`. Slices preserve marks and nested node structure, unlike plain text extraction.

**Called by:** [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] (`duplicateSelection`, `toggleDetails`, `wrapSelectionInDetailsBlockRange`)

### `sliceContentToNodeArray(slice)`

Iterates `slice.content.childCount` and collects children into a plain array. Used when constructing container nodes (details, blockquote wrappers) from selected content.

### `getSelectionDebugInfo(state)`

Returns a structured object with:
- `selection` — type, from, to, empty
- `from`/`to` — pos, depth, parent type
- `nodeRange` — depth, parent type, start, end, indices
- `fromPath`/`toPath` — full ancestor chains with type, start, end, index at each depth

**Called by:** [[Palimpsest-Writing-Software/modules/editor-frontend/ProseMirrorEditor\|ProseMirrorEditor.ts]] (`getSelectionDebugInfo`, `logSelectionDebugInfo`)

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `EditorState` | `prosemirror-state` | State access for selection |
| `Node`, `NodeType`, `NodeRange`, `ResolvedPos`, `Slice` | `prosemirror-model` | Tree traversal types |

## Side Effects

None. Pure utility functions with no state or registrations.
