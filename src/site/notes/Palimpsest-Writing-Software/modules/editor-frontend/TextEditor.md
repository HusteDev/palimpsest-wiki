---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/text-editor/","title":"TextEditor.svelte","tags":["file","editor-frontend","svelte"],"updated":"2026-03-07T17:47:53.964-07:00"}
---


# `TextEditor.svelte`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/TextEditor.svelte`

> [!NOTE] Main Svelte 5 component that mounts the ProseMirror editor, renders the formatting toolbar, manages reactive effects for search/spellcheck/content sync, and coordinates the context menu UI.

## Props

| Prop | Type | Required | Default | Description |
| ---- | ---- | -------- | ------- | ----------- |
| `content` | `string` | no | `''` | Initial HTML content |
| `contentJson` | `Record<string, unknown>` | no | — | ProseMirror JSON content (preferred over HTML) |
| `placeholder` | `string` | no | `'Start writing...'` | Placeholder text |
| `editable` | `boolean` | no | `true` | Whether editing is enabled |
| `showDebug` | `boolean` | no | `false` | Show JSON debug tools (controlled by `dev_tools` setting) |
| `projectSlug` | `string` | no | — | Project slug for spellcheck dictionary and glossary API |
| `projectLanguage` | `string` | no | `'en'` | BCP-47 language tag for Hunspell dictionary |
| `onupdate` | callback | no | — | Called with `{ html, text, json, wordCount, charCount }` on content change |
| `onselection` | callback | no | — | Called with `{ wordCount, charCount }` on selection change |
| `onglossary` | callback | no | — | Called with `{ text }` when user clicks "Add to Glossary" (stub) |
| `oncomment` | callback | no | — | Called with `{ text, pos }` when user clicks "Create Comment" (stub) |

## Reactive State

| Variable | Type | Source | Description |
| -------- | ---- | ------ | ----------- |
| `editor` | `ProseMirrorEditor \| null` | `$state` | Editor instance (created on mount) |
| `spellcheckProvider` | `TextAnalysisProvider \| null` | `$state` | Active spellcheck provider |
| `isInternalUpdate` | `boolean` | `$state` | Guards against re-entrant content updates |
| `canUndo` / `canRedo` | `boolean` | `$state` | Toolbar undo/redo enable state |
| `activeBold`, `activeItalic`, ... | `boolean` | `$state` | 14 toolbar active state flags |
| `contextMenuData` | `ContextMenuData \| null` | `$state` | Context menu display data |
| `lastSetContentJson` | `Record<string, unknown> \| null` | local | Prevents infinite content sync loops |

## Lifecycle

### `onMount`

1. Creates `ProseMirrorEditor` with element, content, callbacks
2. Sets initial `canUndo`/`canRedo` and active states
3. Wires up `GlossaryMarkProvider` (if `projectSlug` is set):
   - `getTermInfo(termId)` → calls `glossaryApi.getEntry()`
   - `navigateToTerm(termId)` → dispatches `glossary:view-term` via `ModuleEventBus`
4. Subscribes to glossary events for tooltip cache invalidation:
   - `glossary:entry-saved` → `clearGlossaryTooltipCache()`
   - `glossary:entry-deleted` → `clearGlossaryTooltipCache()`

### `onDestroy`

1. Unsubscribes from glossary events
2. Destroys spellcheck provider and clears dictionary store
3. Clears glossary mark provider
4. Destroys editor instance

## Reactive Effects (`$effect`)

| Effect | Trigger | Action |
| ------ | ------- | ------ |
| **Content sync** | `contentJson` or `content` prop change | Updates editor content (skips if editor has focus to avoid overwriting while typing) |
| **Editable sync** | `editable` prop change | Calls `editor.setEditable(editable)` |
| **Search highlighting** | `$searchStore.query` | Calls `editor.setSearchHighlights(query)` to update decorations |
| **Native spellcheck suppression** | `editorElement` available | Sets `spellcheck="false"` attribute on the DOM element |
| **Spellcheck toggle** | `$userPreferences.editor.spellcheck` | Creates or destroys the spellcheck provider; loads project dictionary |

## Spellcheck Provider Lifecycle

When spellcheck is **enabled** (preference toggle):

1. `dictionaryStore.loadDictionary(projectSlug)` — loads custom words from backend
2. `createSpellcheckProvider(projectLanguage, customWords)` — creates Hunspell-based provider
3. `dictionaryStore.setProvider(provider)` — wires provider into dictionary store (for add/remove word)
4. `editor.enableSpellcheck(provider)` — dispatches 'enable' meta to textAnnotation plugin

When spellcheck is **disabled**:

1. `editor.disableSpellcheck()` — dispatches 'disable' meta
2. `dictionaryStore.setProvider(null)` — disconnects from dictionary store
3. `spellcheckProvider.destroy()` — releases Hunspell resources

## Toolbar

The component renders a formatting toolbar when `editable` is true, organized into groups:

| Group | Buttons |
| ----- | ------- |
| Inline formatting | Bold, Italic, Underline, Strikethrough, Code |
| Block types | H1, H2, H3, Paragraph |
| Lists & blockquote | Bullet List, Ordered List, Blockquote |
| Actions | Undo, Redo, Horizontal Rule, Table, Image, Highlight, Details |
| Debug (if `showDebug`) | JSON Viewer |

All toolbar actions use the `editor.chain().focus().<command>().run()` pattern.

## Context Menu Integration

1. `ProseMirrorEditor.onContextMenu` callback stores `contextMenuData`
2. If data is non-null and editor is editable, renders [[Palimpsest-Writing-Software/modules/editor-frontend/EditorContextMenu\|EditorContextMenu]] component
3. `handleContextMenuAction(action, data)` processes menu item clicks:
   - Formatting actions → `editor.chain().focus().<command>().run()`
   - Table actions → direct command execution
   - List/blockquote actions → direct command execution
   - `addToDictionary` → `dictionaryStore.addWord()` then `editor.triggerSpellcheck()`
   - `glossary` → emits `onglossary` callback
   - `comment` → emits `oncomment` callback
   - `suggest:<word>` → `editor.replaceWord(from, to, suggestion)`

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `ProseMirrorEditor`, types | `./proseMirror/ProseMirrorEditor` | Editor wrapper |
| `getSelectedText` | `./proseMirror/wordSelection` | Selection text extraction |
| `wordCountService` | `$lib/services` | Word/character counting |
| `searchStore` | `$lib/stores/search` | Search query subscription |
| `dictionaryStore` | `$lib/stores/dictionary` | Custom word management |
| `glossaryApi` | `$lib/api/glossary` | Glossary entry fetch for tooltips |
| `createSpellcheckProvider` | `$lib/services/textAnalysis` | Spellcheck provider factory |
| `userPreferences` | `$lib/stores/preferences` | Spellcheck toggle preference |
| `dispatch`, `on` | `$lib/services/moduleEventBus` | Glossary event dispatch/subscription |
| `EditorContextMenu` | `./EditorContextMenu.svelte` | Context menu component |
| 19 Lucide icons | `lucide-svelte` | Toolbar icons |

## Side Effects

- Mounts ProseMirror EditorView into the DOM
- Subscribes to ModuleEventBus for glossary events
- Modifies DOM attributes (`spellcheck="false"`)
- Creates/destroys spellcheck providers (Hunspell WASM instances)
