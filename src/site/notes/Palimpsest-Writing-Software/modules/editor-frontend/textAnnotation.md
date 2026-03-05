---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/text-annotation/","title":"textAnnotation.ts","tags":["file","editor-frontend","prosemirror","plugin"]}
---


# `textAnnotation.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/plugins/textAnnotation.ts`

> [!NOTE] Generic ProseMirror plugin for rendering text analysis issues (spelling, grammar, style) as visual-only `Decoration.inline()` overlays. Consumes any `TextAnalysisProvider` and follows the PluginKey/state/meta/apply pattern.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `TextAnnotationMeta` | type | Discriminated union for transaction meta actions |
| `textAnnotationKey` | `PluginKey<TextAnnotationState>` | Key for state access and meta dispatch |
| `setTextAnnotationProvider` | function | Set or clear the provider on the CheckManager |
| `getCheckManager` | function | Get the module-level CheckManager instance |
| `createTextAnnotationPlugin` | function | Factory that creates the plugin |

## Plugin Pattern

This plugin follows the same **PluginKey/state/meta/apply** pattern as [[Palimpsest-Writing-Software/modules/search-frontend/searchHighlight\|searchHighlight.ts]]:

1. External code dispatches actions via `tr.setMeta(textAnnotationKey, meta)`
2. The plugin's `apply()` method processes the meta and updates internal state
3. The `props.decorations()` method provides decoration sets to the view

## Meta Actions

| Action | Fields | Description |
| ------ | ------ | ----------- |
| `setIssues` | `issues: TextIssue[]` | Replace all issues with new check results; rebuilds decorations |
| `enable` | — | Enable the plugin (start responding to doc changes) |
| `disable` | — | Disable: clear all issues and decorations, cancel pending checks |
| `clear` | — | Clear issues and decorations without disabling |

## Internal State (`TextAnnotationState`)

| Field | Type | Description |
| ----- | ---- | ----------- |
| `issues` | `TextIssue[]` | All current text analysis issues |
| `decorations` | `DecorationSet` | Inline decorations for rendering issues |
| `enabled` | `boolean` | Whether the plugin is actively checking |

## CheckManager Class

The `CheckManager` manages debounced checking triggered by document changes:

| Method | Description |
| ------ | ----------- |
| `setProvider(provider)` | Set the `TextAnalysisProvider` instance (or null to disable) |
| `getProvider()` | Get the current provider reference |
| `scheduleCheck(view)` | Start a debounced check (cancels any pending check first) |
| `cancel()` | Cancel any pending check timer |
| `destroy()` | Cancel timers and clear provider reference |

### Check Flow

1. `scheduleCheck()` cancels any pending timer and starts a new one (`DEBOUNCE_MS = 400`)
2. Timer fires: `runCheck()` walks all text nodes via `doc.descendants()`
3. For each text node, calls `provider.check(text, pos)` — returns `Promise<TextIssue[]>`
4. All promises are collected via `Promise.all()`
5. Results are dispatched as `{ action: 'setIssues', issues }` meta (with `addToHistory: false`)

> [!IMPORTANT] The CheckManager is a **module-level singleton**, not per-plugin-instance. This allows external code (`ProseMirrorEditor.enableSpellcheck()`) to set the provider without needing a reference to the plugin instance.

## Decoration CSS Classes

| Issue Type | CSS Class | Visual |
| ---------- | --------- | ------ |
| `spelling` | `.spelling-error` | Red wavy underline |
| `grammar` | `.grammar-error` | Blue wavy underline |
| `style` | `.style-warning` | (configured via CSS variables) |

Each decoration also stores `data-issue-type` and `data-issue-word` attributes for context menu access.

## Transaction Apply Logic

| Condition | Action |
| --------- | ------ |
| Meta `setIssues` | Rebuild decorations from new issues |
| Meta `enable` | Set `enabled: true` |
| Meta `disable` | Cancel checks, clear issues and decorations, set `enabled: false` |
| Meta `clear` | Clear issues and decorations, keep `enabled` state |
| `docChanged` and enabled | Map existing decorations (keep roughly correct until re-check) |
| No meta, no doc change | Map decorations through transaction mapping |

## View Lifecycle

The plugin's `view()` method provides:

- **`update(view, prevState)`:** If enabled and document content changed, schedules a re-check via `checkManager.scheduleCheck(view)`
- **`destroy()`:** Cancels any pending check timer

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Plugin`, `PluginKey` | `prosemirror-state` | Plugin factory and key |
| `Decoration`, `DecorationSet` | `prosemirror-view` | Decoration creation |
| `EditorState`, `Transaction` | `prosemirror-state` | State types |
| `EditorView` | `prosemirror-view` | View for check scheduling |
| `Node` | `prosemirror-model` | Document traversal |
| `TextAnalysisProvider`, `TextIssue` | `$lib/services/textAnalysis/types` | Provider interface and issue type |

## Side Effects

- Module-level `CheckManager` singleton persists across plugin instances
- `scheduleCheck()` sets a `setTimeout` that dispatches a transaction when it fires
- `addToHistory: false` meta prevents spellcheck results from entering the undo stack
