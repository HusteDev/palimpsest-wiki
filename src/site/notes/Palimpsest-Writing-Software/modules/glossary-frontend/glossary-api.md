---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-frontend/glossary-api/","title":"glossary.ts (API Client)","tags":["file","glossary-frontend"],"updated":"2026-03-05T05:32:42.823-07:00"}
---


# `glossary.ts (API Client)`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend]]
**Path:** `src/lib/api/glossary.ts`

> [!NOTE] HTTP client module providing typed methods for all 6 glossary API endpoints. Uses the shared apiClient from `./client` and unwraps APIResponse<T> wrappers.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `listEntries` | function | GET list with filter params |
| `getEntry` | function | GET single entry by ID |
| `createEntry` | function | POST new entry |
| `updateEntry` | function | PUT partial update |
| `deleteEntry` | function | DELETE entry |
| `getOccurrences` | function | GET term occurrences in content |

All six functions are grouped under a single named export `glossaryApi` object, matching the `contentApi` and `searchApi` patterns used elsewhere.

The module also re-exports the following types from `$lib/types/glossary` for consumer convenience: `GlossaryEntry`, `GlossaryTermStatus`, `TermScope`, `SpoilerLevel`, `GlossaryCategory`, `PartOfSpeech`, `GlossarySortBy`, `SortOrder`, `CreateGlossaryEntryInput`, `UpdateGlossaryEntryInput`, `ListGlossaryEntriesResponse`, `ListGlossaryEntriesParams`, `TermOccurrence`, `TermOccurrencesResponse`.

## Key Methods

### `listEntries(projectSlug: string, params?: ListGlossaryEntriesParams): Promise<ListGlossaryEntriesResponse>`

- **Endpoint:** GET `/api/projects/{slug}/glossary`
- **Params:** `limit`, `offset`, `search`, `tags` (joined with commas), `category`, `status`, `sortBy`, `sortOrder`
- **Returns:** `{ entries: GlossaryEntry[], total: number, limit: number, offset: number }`
- Builds a `URLSearchParams` from non-undefined param values and appends as query string.

### `getEntry(projectSlug: string, entryId: string): Promise<GlossaryEntry>`

- **Endpoint:** GET `/api/projects/{slug}/glossary/{id}`
- **Returns:** A single `GlossaryEntry` object.
- Throws if the entry is not found or the API returns `success: false`.

### `createEntry(projectSlug: string, input: CreateGlossaryEntryInput): Promise<GlossaryEntry>`

- **Endpoint:** POST `/api/projects/{slug}/glossary`
- **Body:** JSON with `term`, `shortDefinition` + optional fields (`longDefinition`, `tags`, `category`, `status`, `partOfSpeech`, `aliases`, `scope`, `spoilerLevel`, `autoMark`, `enableTooltip`, `enableHyperlink`)
- **Returns:** The newly created `GlossaryEntry`.
- Throws on validation failure (empty term/definition, duplicate term).

### `updateEntry(projectSlug: string, entryId: string, input: UpdateGlossaryEntryInput): Promise<GlossaryEntry>`

- **Endpoint:** PUT `/api/projects/{slug}/glossary/{id}`
- **Body:** JSON with only non-null changed fields (all fields optional, at least one required by backend).
- **Returns:** The updated `GlossaryEntry`.
- Throws if the entry is not found, validation fails, or duplicate term.

### `deleteEntry(projectSlug: string, entryId: string): Promise<void>`

- **Endpoint:** DELETE `/api/projects/{slug}/glossary/{id}`
- **Returns:** Nothing. Backend returns `204 No Content` on success.
- Throws if the entry is not found or the API call fails.

### `getOccurrences(projectSlug: string, entryId: string): Promise<TermOccurrencesResponse>`

- **Endpoint:** GET `/api/projects/{slug}/glossary/{id}/occurrences`
- **Returns:** `TermOccurrencesResponse` with matching content documents found via FTS5 search.
- Throws if the entry is not found or the API call fails.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `apiClient` | `./client` | Shared HTTP client with base URL and error handling |
| `APIResponse<T>` | `$lib/types/project` | Response wrapper type used to unwrap `success` / `data` / `error` |
| `GlossaryEntry` | `$lib/types/glossary` | TypeScript interface for a glossary entry |
| `CreateGlossaryEntryInput` | `$lib/types/glossary` | Input type for creating entries |
| `UpdateGlossaryEntryInput` | `$lib/types/glossary` | Input type for updating entries (all fields optional) |
| `ListGlossaryEntriesResponse` | `$lib/types/glossary` | Paginated list response type |
| `ListGlossaryEntriesParams` | `$lib/types/glossary` | Query parameter type for list filtering |
| `TermOccurrencesResponse` | `$lib/types/glossary` | Response type for term occurrence search |

## Side Effects

- Network I/O (HTTP requests). Every method makes at least one HTTP call to the backend API.
- All methods throw on API failure (non-success response or missing data), except `deleteEntry` which calls `apiClient.delete` directly without unwrapping.
