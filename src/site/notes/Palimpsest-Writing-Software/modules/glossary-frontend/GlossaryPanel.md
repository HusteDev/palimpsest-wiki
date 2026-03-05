---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-frontend/glossary-panel/","title":"GlossaryPanel.svelte","tags":["file","glossary-frontend"]}
---


# GlossaryPanel.svelte

> [!NOTE] Root container component that acts as a state-machine view router, switching child components based on glossaryStore.viewMode.

**Source:** `src/lib/components/glossary/GlossaryPanel.svelte`

## Exports

| Export | Kind | Description |
| ------ | ---- | ----------- |
| GlossaryPanel | default Svelte component | State-machine view router for the glossary sidebar panel |

## State Machine

The panel renders exactly one child component at a time, determined by the `viewMode` value in [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]]:

| viewMode | Component Rendered | Trigger |
| -------- | ------------------ | ------- |
| `'list'` | GlossaryList | Default state, after delete completes |
| `'detail'` | GlossaryEntryDetail | Click entry in list, `glossary:view-term` event |
| `'create'` | GlossaryEntryForm (mode=create) | "New" button in list header, context menu "Add to Glossary" |
| `'edit'` | GlossaryEntryForm (mode=edit) | "Edit" button from detail view |

## Lifecycle

### onMount

1. Subscribes to the `glossary:view-term` event on moduleEventBus. When received, calls `glossaryStore.navigateToEntry()` with the event payload's entry ID to switch to detail view for the referenced term.
2. Calls `glossaryStore.loadEntries(projectSlug)` if `projectSlug` is set, triggering the initial fetch of glossary entries for the current project.

### onDestroy

1. Unsubscribes from all moduleEventBus event listeners registered during mount.
2. Clears store state to prevent stale data from persisting when the panel is unmounted.

## Dependencies

| Dependency | Type | Purpose |
| ---------- | ---- | ------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]] | internal | Reactive state — viewMode, entries, selected entry |
| moduleEventBus | internal | Subscribe to `glossary:view-term` events |
| GlossaryList | child component | List view rendering |
| GlossaryEntryDetail | child component | Detail view rendering |
| GlossaryEntryForm | child component | Create/edit form rendering |

## Side Effects

- Subscribes to `moduleEventBus` on mount; unsubscribes on destroy.
- Triggers network I/O indirectly via `glossaryStore.loadEntries()` on mount.
