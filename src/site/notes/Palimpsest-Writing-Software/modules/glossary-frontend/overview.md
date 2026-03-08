---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-frontend/overview/","title":"Glossary Frontend Module","tags":["module","glossary","frontend"],"updated":"2026-03-05T05:32:43.361-07:00"}
---


# Glossary Frontend Module

> [!NOTE] Svelte 5 frontend module providing the glossary sidebar panel, entry management components, reactive store, API client, and ProseMirror editor integration.

## Responsibilities

- Sidebar panel with list / detail / create / edit views
- Reactive state management via [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]]
- REST API client for all 6 glossary endpoints
- ProseMirror [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|glossaryLink mark plugin]] (tooltip + click navigation)
- Event bus integration for cross-module communication
- Context menu "Add to Glossary" from editor text selection
- Spellcheck dictionary integration for invented terms

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/GlossaryPanel\|GlossaryPanel]] | component | State-machine view router |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/GlossaryEntryForm\|GlossaryEntryForm]] | component | Create/edit form with 8 sections |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]] | store | Reactive state and API methods |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-api\|glossaryApi]] | module | HTTP client for glossary endpoints |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossaryMark\|glossaryMark plugin]] | plugin | ProseMirror tooltip + navigation |

## Internal Structure — Component Tree

```
GlossaryPanel.svelte
  |-- GlossaryList.svelte           (viewMode='list')
  |     +-- GlossaryEntryItem.svelte  (per entry row)
  |-- GlossaryEntryDetail.svelte    (viewMode='detail')
  |-- GlossaryEntryForm.svelte      (viewMode='create' | 'edit')
  |     |-- GlossaryTermPicker.svelte  (x6, per relationship field)
  |     +-- ScopeTreePicker.svelte
  |           +-- CheckboxTree.svelte
  |                 +-- CheckboxTreeNode.svelte  (recursive)
```

### GlossaryList

Provides the main entry listing with debounced search (300ms), category/status filters, A-Z alphabet nav, keyboard navigation, and Load More pagination. The search input triggers `glossaryStore.setSearchFilter()` after the debounce period. Category and status dropdowns call their respective store filter methods. The A-Z alphabet bar scrolls to entries starting with the selected letter. Arrow-key navigation moves focus between entry rows; Enter selects the focused entry.

### GlossaryEntryItem

Pure presentational component rendering a single entry row. Displays a status dot (color-coded by entry status), the term name, a category badge, a truncated definition (first 80 characters of shortDefinition), alias count indicator, and tag pills. Emits a click event for selection — all data logic is handled by the parent GlossaryList.

### GlossaryEntryDetail

Full entry display with all metadata fields rendered in organized sections. Provides edit and delete action buttons in the header. Includes a "Find in Content" feature that calls the occurrences API endpoint and displays results as a clickable list of content documents with occurrence counts. Relationship chips (broaderTerms, narrowerTerms, relatedTerms, synonyms, antonyms, seeAlso) are rendered as clickable pills that navigate to the referenced entry. Usage count auto-reloads 4 seconds after a `glossary:entry-saved` event to reflect updated term scan results.

### GlossaryTermPicker

Search-and-select component used for all 6 relationship fields in the entry form. Features debounced search (300ms, minimum 2 characters) against the glossary API, displays results in a dropdown, and renders selected terms as removable chips. Self-reference exclusion prevents an entry from being added to its own relationship fields. Supports keyboard navigation (arrow keys to move, Enter to select, Escape to close dropdown).

### ScopeTreePicker

Scope selection component with a "Full Project" checkbox at the top and a CheckboxTree below populated from the content API. When "Full Project" is checked, the tree is disabled and `scopeTargets` is set to an empty array (meaning all content). When unchecked, users can select individual content items to scope the glossary entry.

### CheckboxTree / CheckboxTreeNode

Generic reusable tree component with checkbox selection at each node. CheckboxTreeNode renders recursively for nested content structures. Supports ancestor-aware selection (checking a parent checks all children; unchecking a child unchecks all ancestors). Auto-expands tree paths that contain pre-selected nodes when the component mounts, ensuring visibility of existing scope selections during edit mode.

## Panel Registration

```typescript
register({
    id: 'glossary', name: 'Glossary', icon: 'BookOpen',
    tier: 'core', alwaysEnabled: true,
    defaultPosition: 'right', allowedPositions: ['left', 'right', 'bottom'],
    defaultEnabled: true, order: 5, source: 'builtin',
    component: () => import('$lib/components/glossary/GlossaryPanel.svelte')
});
```

The glossary panel is registered as a Core-tier built-in panel. It is always enabled and cannot be disabled by users. The default position is the right sidebar, but users can move it to the left sidebar or bottom panel area. Load order 5 places it after the content navigator and before optional panels.

## Dependencies

| Dependency | Type | Purpose |
| ---------- | ---- | ------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-api\|glossaryApi]] | internal | REST API client |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]] | internal | Reactive state management |
| moduleEventBus | internal | Cross-module event system |
| dictionaryStore | internal | Spellcheck dictionary |
| ProseMirror | external | Editor framework |
| Lucide icons | external | Icon library |

## Dataflow Diagrams

Detailed dataflow diagrams are available on the feature canvases:

- [[Palimpsest-Writing-Software/features/glossary/diagrams/create-entry-flow\|Create Entry: Full Flow]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/read-entries-flow\|Read Entries: List, Get, and Occurrences]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/update-entry-flow\|Update Entry: With Reciprocal Diffs]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/delete-entry-flow\|Delete Entry: Cleanup and Mark Removal]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/editor-integration-flow\|Editor Integration: Tooltip, Click, Context Menu]]

## Related

- [[Palimpsest-Writing-Software/features/glossary/overview\|Feature: Glossary System]]
- [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Module: Glossary Backend]]
- [[Palimpsest-Writing-Software/api/glossary/overview\|API: Glossary Service]]
- [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|Data Model: GlossaryEntry]]
- [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|Data Model: GlossaryEntryDTO]]
