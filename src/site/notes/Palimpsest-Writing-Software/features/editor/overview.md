---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/editor/overview/","title":"ProseMirror Editor","tags":["feature","editor","prosemirror","core"],"updated":"2026-03-05T06:38:38.243-07:00"}
---


# ProseMirror Editor

> [!NOTE] WYSIWYG rich text editor built directly on the ProseMirror framework (no TipTap wrapper). Provides inline formatting, block structures, tables, images, and integrates with the Glossary, Search, and Spellcheck systems via a plugin architecture.

## Overview

The ProseMirror Editor is the central authoring surface of the Palimpsest application. Authors write and edit content inside a ProseMirror `EditorView` that supports:

- **Inline formatting:** bold, italic, underline, strikethrough, inline code, highlight
- **Block structures:** paragraphs, headings (H1-H3), bullet lists, ordered lists, blockquotes, details/summary
- **Tables:** full table editing with header rows, column resizing, row/column add/delete, cell merge/split
- **Images:** inline images via drag-and-drop, clipboard paste, or URL insertion (base64 data URL)
- **Glossary marks:** backend-applied `glossaryLink` marks rendered with hover tooltips and optional click-to-navigate
- **Search highlighting:** visual-only decorations for search query matches (never serialized)
- **Spellcheck:** visual-only decorations for spelling/grammar/style issues via a pluggable `TextAnalysisProvider`
- **Context menu:** right-click context menu with smart word selection, context-aware actions, and spellcheck integration

## Architecture

The editor follows a layered architecture:

```
Route Layer          +page.svelte — content loading, persistence, draft lifecycle
     |
Component Layer      TextEditor.svelte — toolbar, reactive effects, lifecycle
     |
Wrapper Layer        ProseMirrorEditor.ts — clean API, plugin assembly, serialization
     |
Core PM Layer        schema.ts + commands.ts + pmUtils.ts + wordSelection.ts
     |
Plugin Layer         searchHighlight | textAnnotation | glossaryMark | contextMenu | fileHandler
```

### Layer Responsibilities

| Layer | Responsibility |
| ----- | -------------- |
| **Route** | Loads project/content from backend, manages two-tier persistence (localStorage draft + backend sync), handles unsaved changes modal |
| **Component** | Mounts ProseMirrorEditor, renders formatting toolbar, wires reactive effects (search, spellcheck, content sync), manages context menu UI |
| **Wrapper** | Creates EditorState + EditorView, assembles plugins in correct order, exposes clean public API (serialization, command chain, search, spellcheck, glossary) |
| **Core** | Document schema definition, 40+ formatting/structural commands, selection utilities, word boundary detection |
| **Plugin** | Self-contained ProseMirror plugins using PluginKey/state/meta/apply or DOM event handling patterns |

## Document Schema

The schema is defined in [[Palimpsest-Writing-Software/modules/editor-frontend/schema\|schema.ts]] and extends ProseMirror's basic nodes and marks.

### Nodes

| Node | Group | Source | Description |
| ---- | ----- | ------ | ----------- |
| `doc`, `paragraph`, `text`, `blockquote`, `heading`, `horizontal_rule`, `code_block`, `hard_break` | block/inline | `prosemirror-schema-basic` | Standard document structure |
| `bullet_list`, `ordered_list`, `list_item` | block | `prosemirror-schema-list` | List structures |
| `table`, `table_row`, `table_cell`, `table_header` | block | `prosemirror-tables` | Table structures with cell background attribute |
| `details` | block | custom | Collapsible container (`content: 'summary block*'`) |
| `summary` | — | custom | Summary line inside details (`content: 'inline*'`) |
| `image` | inline | custom | Inline draggable image with `src`, `alt`, `title` attrs |

### Marks

| Mark | Source | Attrs | Description |
| ---- | ------ | ----- | ----------- |
| `strong`, `em`, `code`, `link` | `prosemirror-schema-basic` | standard | Basic inline formatting |
| `strikethrough` | custom | none | `<s>` / `<del>` / `<strike>` / `text-decoration: line-through` |
| `underline` | custom | none | `<u>` / `text-decoration: underline` |
| `highlight` | custom | `color` | `<mark>` with optional `data-color` attribute |
| `glossaryLink` | custom | `termId`, `termSlug`, `color`, `hoverColor`, `enableHyperlink` | Applied by backend TermScanHandler; `inclusive: false` prevents mark extension at edges |

## Plugin Architecture

All plugins are assembled in [[Palimpsest-Writing-Software/modules/editor-frontend/ProseMirrorEditor\|ProseMirrorEditor.ts]] in a specific order:

| Order | Plugin | Pattern | Serialized | Description |
| ----- | ------ | ------- | ---------- | ----------- |
| 1 | `history()` | built-in | no | Undo/redo stack |
| 2 | `keymap({...})` | built-in | no | Keyboard shortcuts for marks and history |
| 3 | `keymap(baseKeymap)` | built-in | no | Default ProseMirror keybindings |
| 4 | `columnResizing()` | built-in | no | Table column drag handles |
| 5 | `tableEditing()` | built-in | no | Table cell selection and editing |
| 6 | `dropCursor()` | built-in | no | Visual cursor for drag-and-drop positioning |
| 7 | `gapCursor()` | built-in | no | Cursor for positions between block nodes |
| 8 | `fileHandler()` | DOM events | no | Image drag-and-drop and paste handling |
| 9 | `contextMenu` | DOM events | no | Right-click context menu with word selection |
| 10 | placeholder | decoration | no | Empty document placeholder text |
| 11 | `searchHighlight` | PluginKey/meta/apply | **never** | Search match decorations |
| 12 | `textAnnotation` | PluginKey/meta/apply | **never** | Spellcheck/grammar decorations |
| 13 | `glossaryMark` | DOM events | **marks are** | Tooltip and click for glossary marks |

> [!IMPORTANT] Plugins 11 and 12 use `Decoration.inline()` overlays that are **never** serialized to the document. They exist purely as visual feedback. Plugin 13 handles marks that **are** part of the document model and are serialized on save.

## Command System

The editor provides 40+ commands in [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]], organized by category:

- **Mark toggles:** `toggleBold`, `toggleItalic`, `toggleCode`, `toggleStrikethrough`, `toggleUnderline`, `toggleHighlight`
- **Block types:** `setParagraph`, `setHeading(level)`
- **Lists:** `toggleBulletList`, `toggleOrderedList`, `splitListItemCommand`, `liftListItemCommand`, `sinkListItemCommand`
- **Wrapping:** `toggleBlockquote`, `toggleDetails`, `wrapInBlockquoteManual`, `wrapSelectionInDetailsBlockRange`
- **Insertion:** `insertHorizontalRule`, `insertImage(src, alt, title)`, `insertTable(rows, cols, withHeaderRow)`
- **Structural:** `duplicateSelection`
- **Table:** 8 re-exported commands from `prosemirror-tables`
- **History:** `undoCommand`, `redoCommand`
- **State helpers:** `isMarkActive`, `isNodeActive`
- **Debug:** `logSelectionDebugInfo`, `toggleEntityTag`

All commands follow the standard ProseMirror `(state, dispatch?, view?) => boolean` signature via the `fromPMCommand` adapter.

### CommandChain Fluent API

```ts
editor.chain().focus().toggleBold().run()
```

The [[Palimpsest-Writing-Software/modules/editor-frontend/commands#CommandChain\|CommandChain]] class provides a fluent builder for composing commands. Critically, it **re-reads `view.state`** before each command in the chain, ensuring subsequent commands see changes made by earlier ones.

## Persistence

Content uses a two-tier persistence model managed by the route layer (`+page.svelte`):

| Tier | Storage | Trigger | Latency |
| ---- | ------- | ------- | ------- |
| Draft | `localStorage` | Debounced on content change (3s) | Immediate |
| Sync | Backend REST API (SQLite) | Manual `Ctrl+S` or auto-save | Network round-trip |

Draft status lifecycle: `idle` -> `saving` -> `draft_saved` -> `synced`

## Integration Points

### Glossary System

- Backend `TermScanHandler` applies `glossaryLink` marks to content
- [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|glossaryMark plugin]] handles hover tooltips and click-to-navigate
- `TextEditor.svelte` sets up the `GlossaryMarkProvider` with API fetch and `ModuleEventBus` navigation
- Tooltip cache is cleared on `glossary:entry-saved` and `glossary:entry-deleted` events

### Search System

- [[Palimpsest-Writing-Software/modules/search-frontend/searchHighlight\|searchHighlight plugin]] renders visual-only decorations for search matches
- `TextEditor.svelte` subscribes to `searchStore.query` and dispatches highlight updates via reactive `$effect`
- Decorations use CSS class `.search-highlight` and are never serialized

### Spellcheck System

- [[Palimpsest-Writing-Software/modules/editor-frontend/textAnnotation\|textAnnotation plugin]] renders visual-only decorations for spelling/grammar issues
- `TextEditor.svelte` manages provider lifecycle (create/destroy) based on `$userPreferences.editor.spellcheck`
- `EditorContextMenu` shows spelling suggestions and "Add to Dictionary" action
- Dictionary store persists custom words per project

## Design Decisions

| Decision | Rationale |
| -------- | --------- |
| Direct ProseMirror (no TipTap) | Full control over plugin pipeline, schema, and rendering; avoids TipTap's abstraction overhead and opinionated extension system |
| Decoration-only overlays for search/spellcheck | Cleanly separates ephemeral visual feedback from persistent document content; prevents accidental serialization |
| `inclusive: false` on glossaryLink | Prevents typing at mark edges from extending the glossary link to new text |
| Base64 data URLs for images | Avoids external file references during Core license (local-only); Pro license will add cloud storage |
| Placeholder via widget decoration | Visual-only UI state that does not enter the document model |
| CommandChain state refresh | Each chained command re-reads `view.state` to see effects of the previous command; prevents stale state bugs with storedMarks and resolved positions |

## Security

- No raw HTML injection — all content passes through ProseMirror's `DOMParser.fromSchema()` which validates against the schema
- `glossaryLink` mark attributes (`termId`, `termSlug`) are sanitized via DOM attribute extraction (no script injection)
- Image `src` is rendered as-is (base64 data URL); external URLs should be validated before insertion (not yet enforced)
- File handler filters uploads by MIME type allowlist: `image/png`, `image/jpeg`, `image/gif`, `image/webp`, `image/svg+xml`

## Performance

- **Search highlighting:** Full document scan on query change; for very large documents, consider future optimization with position-based scanning
- **Spellcheck:** Debounced at 400ms after document changes; walks all text nodes and calls provider per node; existing decorations are mapped (not rebuilt) during the debounce window
- **Toolbar state updates:** `updateActiveStates()` runs on every transaction (including selection-only changes); this is fast (mark/node type lookups) but scales with toolbar item count
- **Glossary tooltips:** Lazy fetch on first hover with in-memory `Map` cache; cache is cleared on glossary data changes, not on a timer

## Logging

| Logger Name | File | Purpose |
| ----------- | ---- | ------- |
| `prosemirror` | `ProseMirrorEditor.ts` | Parse errors, spellcheck lifecycle, glossary provider lifecycle |
| `pm-commands` | `commands.ts` | Selection debug info |
| `search-highlight` | `searchHighlight.ts` | Match count per scan, stub action invocations |
| `file-handler` | `fileHandler.ts` | Image processing errors |
| `context-menu` | `EditorContextMenu.svelte` | Context menu action dispatch |

## Dataflow Diagrams

- [[Palimpsest-Writing-Software/features/editor/diagrams/editor-init-flow\|Editor Initialization: Mount to Ready]]
- [[Palimpsest-Writing-Software/features/editor/diagrams/content-save-flow\|Content Editing: Change to Persist]]
- [[Palimpsest-Writing-Software/features/editor/diagrams/context-menu-flow\|Context Menu: Right-Click to Action]]

## Related

- [[Palimpsest-Writing-Software/_index/moc-core-features\|MOC: Core Features]]
- [[Palimpsest-Writing-Software/features/glossary/overview\|Feature: Glossary System]]
- [[Palimpsest-Writing-Software/features/search/overview\|Feature: Search Engine]]
- [[Palimpsest-Writing-Software/features/panel-system/overview\|Feature: Panel System]]
