---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-frontend/glossary-store/","title":"glossary.ts (Store)","tags":["file","glossary-frontend"]}
---


# glossary.ts (Store)

> [!NOTE] Reactive Svelte store managing glossary panel state, entry data, filters, pagination, and view mode transitions. All component data flows through this store.

**Source:** `src/lib/stores/glossary.ts`

## Exports

| Export | Kind | Description |
| ------ | ---- | ----------- |
| glossaryStore | singleton store instance | Reactive state and API methods for the glossary panel |

## State Shape

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| entries | `GlossaryEntry[]` | `[]` | Current page of entries loaded from the API |
| total | `number` | `0` | Total count of matching entries for pagination |
| isLoading | `boolean` | `false` | Whether an API call is currently in flight |
| error | `string \| null` | `null` | Last error message from a failed API call |
| selectedEntryId | `string \| null` | `null` | ID of the currently selected entry |
| selectedEntry | `GlossaryEntry \| null` | `null` | Full data of the currently selected entry |
| searchFilter | `string` | `""` | Current search text applied to entry listing |
| tagFilter | `string[]` | `[]` | Active tag filters applied to entry listing |
| categoryFilter | `string` | `""` | Active category filter applied to entry listing |
| statusFilter | `string` | `""` | Active status filter applied to entry listing |
| currentOffset | `number` | `0` | Pagination offset for the current entry listing |
| limit | `number` | `50` | Number of entries fetched per page |
| projectSlug | `string` | `""` | Slug of the current project |
| viewMode | `GlossaryViewMode` | `'list'` | Current panel view: list, detail, create, or edit |
| pendingTerm | `string` | `""` | Pre-populated term for create mode (from context menu) |

## Methods

| Method | Description |
| ------ | ----------- |
| `loadEntries(projectSlug)` | Fetch entries from the API using current filters. Resets offset to 0 and replaces the entries array. Sets isLoading during the call and captures errors. |
| `loadMore()` | Increment offset by limit and fetch the next page. Appends results to the existing entries array for infinite-scroll pagination. |
| `selectEntry(projectSlug, entryId)` | Fetch the full entry by ID from the API. Sets selectedEntryId and selectedEntry, then switches viewMode to `'detail'`. |
| `createEntry(projectSlug, input)` | Send create request via glossaryApi. Prepend the new entry to the entries array, set it as selectedEntry, and switch viewMode to `'detail'`. |
| `updateEntry(projectSlug, entryId, input)` | Send update request via glossaryApi. Replace the entry in the entries array with the updated version. Update selectedEntry if it matches. |
| `deleteEntry(projectSlug, entryId)` | Send delete request via glossaryApi. Remove the entry from the entries array. Dispatch `glossary:entry-deleted` event. Switch viewMode to `'list'`. |
| `setSearchFilter(text)` | Update the searchFilter state and trigger a reload of entries with the new filter applied. |
| `setTagFilter(tags)` | Update the tagFilter array and trigger a reload of entries with the new tags applied. |
| `setCategoryFilter(category)` | Update the categoryFilter state and trigger a reload of entries with the new category applied. |
| `setStatusFilter(status)` | Update the statusFilter state and trigger a reload of entries with the new status applied. |
| `navigateToEntry(projectSlug, entryId)` | Ensure the glossary panel is visible, fetch the full entry, and switch to detail view. Used by external callers (e.g., ProseMirror mark click, event bus). |
| `startCreate(projectSlug, term?)` | Ensure the glossary panel is visible. Set pendingTerm if provided (from context menu selection). Switch viewMode to `'create'`. |
| `ensureGlossaryPanelVisible()` | Scan userPreferences for the glossary panel. If not present in any panel position, add it to the right panel array and set it as the active panel. |

## Dependencies

| Dependency | Type | Purpose |
| ---------- | ---- | ------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-api\|glossaryApi]] | internal | All HTTP operations for glossary CRUD and listing |
| moduleEventBus | internal | Dispatch `glossary:entry-deleted` event |
| userPreferences | internal | Read/write panel visibility state |

## Side Effects

- Calls glossaryApi methods, which perform network I/O (HTTP requests to the backend).
- Dispatches `glossary:entry-deleted` event on moduleEventBus when an entry is deleted.
- Modifies userPreferences store to ensure the glossary panel is visible when `ensureGlossaryPanelVisible()` is called.
- Shows notification toasts on successful create, update, and delete operations.
