---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/overview/","title":"Editor Frontend Module","tags":["module","editor-frontend"],"updated":"2026-03-07T18:44:50.128-07:00"}
---


# Editor Frontend

> [!NOTE] ProseMirror-based rich text editor module providing document schema, command system, plugin architecture, and Svelte UI components for the authoring surface.

## Responsibilities

- Define the document schema (nodes and marks) that governs legal document structure
- Provide 40+ formatting and structural commands with a fluent CommandChain API
- Assemble and manage ProseMirror plugins (history, keymaps, tables, cursors, search, spellcheck, glossary, context menu, file handling)
- Expose clean serialization API (HTML, plain text, ProseMirror JSON)
- Render the formatting toolbar and context menu UI via Svelte 5 components
- Integrate with Glossary, Search, and Spellcheck systems via plugin providers

## Public API Surface

| Export | Kind | Source File | Description |
| ------ | ---- | ----------- | ----------- |
| `schema` | const | [[Palimpsest-Writing-Software/modules/editor-frontend/schema\|schema.ts]] | ProseMirror `Schema` instance with all nodes and marks |
| `ProseMirrorEditor` | class | [[Palimpsest-Writing-Software/modules/editor-frontend/ProseMirrorEditor\|ProseMirrorEditor.ts]] | Main editor wrapper with lifecycle, serialization, and plugin API |
| `ProseMirrorEditorOptions` | type | [[Palimpsest-Writing-Software/modules/editor-frontend/ProseMirrorEditor\|ProseMirrorEditor.ts]] | Constructor options interface |
| `CommandChain` | class | [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] | Fluent command composition with inter-command state refresh |
| `Command` | type | [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] | `(state, dispatch?, view?) => boolean` command signature |
| `toggleBold`, `toggleItalic`, ... | const | [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] | 6 mark toggle commands |
| `setParagraph`, `setHeading`, ... | const/fn | [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] | Block type and structural commands |
| `isMarkActive`, `isNodeActive` | function | [[Palimpsest-Writing-Software/modules/editor-frontend/commands\|commands.ts]] | Toolbar active state helpers |
| `findWordAtPos`, `selectWordAtPos`, `getSelectedText` | function | [[Palimpsest-Writing-Software/modules/editor-frontend/wordSelection\|wordSelection.ts]] | Unicode-aware word boundary detection |
| `findNearestAncestorOfType`, `isInsideNodeType`, `getBlockRange`, ... | function | [[Palimpsest-Writing-Software/modules/editor-frontend/pmUtils\|pmUtils.ts]] | ResolvedPos/NodeRange/Slice utilities |
| `searchHighlightKey`, `createSearchHighlightPlugin` | const/fn | [[Palimpsest-Writing-Software/modules/search-frontend/searchHighlight\|searchHighlight.ts]] | Search match decoration plugin (documented in search-frontend) |
| `textAnnotationKey`, `createTextAnnotationPlugin`, ... | const/fn | [[Palimpsest-Writing-Software/modules/editor-frontend/textAnnotation\|textAnnotation.ts]] | Spellcheck/grammar decoration plugin |
| `createGlossaryMarkPlugin`, `setGlossaryMarkProvider`, ... | fn | [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|glossaryMark.ts]] | Glossary tooltip/click plugin (documented in glossary-frontend) |
| `createContextMenuPlugin` | fn | [[Palimpsest-Writing-Software/modules/editor-frontend/contextMenu\|contextMenu.ts]] | Right-click context menu plugin |
| `fileHandler` | fn | [[Palimpsest-Writing-Software/modules/editor-frontend/fileHandler\|fileHandler.ts]] | Image drag-and-drop and paste plugin |

## Internal Structure

The module is organized in two layers within `src/lib/components/editor/`:

```
editor/
├── TextEditor.svelte              # Main Svelte component (toolbar, lifecycle, effects)
├── EditorContextMenu.svelte       # Context menu component (positioned popup)
└── proseMirror/
    ├── schema.ts                  # Document schema definition
    ├── ProseMirrorEditor.ts       # Editor wrapper class
    ├── commands.ts                # 40+ commands + CommandChain
    ├── pmUtils.ts                 # ResolvedPos/NodeRange/Slice utilities
    ├── wordSelection.ts           # Word boundary detection
    ├── index.ts                   # Barrel exports
    └── plugins/
        ├── searchHighlight.ts     # Search decoration plugin
        ├── textAnnotation.ts      # Spellcheck decoration plugin
        ├── glossaryMark.ts        # Glossary tooltip/click plugin
        ├── contextMenu.ts         # Context menu plugin
        └── fileHandler.ts         # Image file handling plugin
```

> [!TIP] The `searchHighlight.ts` and `glossaryMark.ts` plugins are physically located in the editor module but are **documented** in their respective feature modules ([[Palimpsest-Writing-Software/modules/search-frontend/searchHighlight\|search-frontend]] and [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|glossary-frontend]]) because they are conceptually owned by those features. Cross-links are provided here.

## Dependencies

| Dependency | Internal/External | Purpose |
| ---------- | ----------------- | ------- |
| `prosemirror-model` | external | Schema, Node, Mark, DOMParser, DOMSerializer |
| `prosemirror-state` | external | EditorState, Transaction, Plugin, PluginKey, TextSelection, NodeSelection |
| `prosemirror-view` | external | EditorView, Decoration, DecorationSet |
| `prosemirror-commands` | external | toggleMark, setBlockType, wrapIn, lift, baseKeymap |
| `prosemirror-keymap` | external | keymap() plugin factory |
| `prosemirror-history` | external | history(), undo, redo |
| `prosemirror-schema-basic` | external | Basic node and mark specs |
| `prosemirror-schema-list` | external | List node specs and commands |
| `prosemirror-tables` | external | Table node specs, editing, column resize, and 8 table commands |
| `prosemirror-transform` | external | findWrapping for structural transforms |
| `prosemirror-inputrules` | external | Input rule support (available but not currently used) |
| `prosemirror-dropcursor` | external | Visual drop cursor |
| `prosemirror-gapcursor` | external | Gap cursor for block boundaries |
| `orderedmap` | external | OrderedMap for schema node composition |
| `[[modules/search-frontend/overview\|search-frontend]]` | internal | `searchStore` for reactive search query subscription |
| `[[modules/glossary-frontend/overview\|glossary-frontend]]` | internal | `glossaryApi` for term data fetch; `glossary-store` for event subscriptions |
| `$lib/services/loggerService` | internal | `createLogger()` for structured logging |
| `$lib/services/wordCountService` | internal | `countAll()` for word/character count |
| `$lib/services/textAnalysis` | internal | `createSpellcheckProvider()` and `TextAnalysisProvider` type |
| `$lib/stores/preferences` | internal | `userPreferences` for spellcheck toggle |
| `$lib/stores/dictionary` | internal | `dictionaryStore` for custom word management |
| `$lib/services/moduleEventBus` | internal | `dispatch`/`on` for glossary events |
| `lucide-svelte` | external | Toolbar and context menu icons |

## Dataflow Diagrams


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/editor/diagrams/editor-init-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Editor Initialization: Mount to Ready

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/editor/diagrams/content-save-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Content Editing: Change to Persist

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/editor/diagrams/context-menu-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Context Menu: Right-Click to Action

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Related

- [[Palimpsest-Writing-Software/features/editor/overview\|Feature: ProseMirror Editor]]
- [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend Module]]
- [[Palimpsest-Writing-Software/modules/search-frontend/overview\|Search Frontend Module]]
- [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend Module]]
