---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-backend/glossary-go/","title":"sqlite/glossary.go","tags":["file","glossary-backend"]}
---


# `sqlite/glossary.go`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Glossary Backend]]
**Path:** `backend/internal/repository/sqlite/glossary.go`

> [!NOTE] SQLite repository implementation for glossary CRUD operations. Handles slug generation, FTS5 indexing, reciprocal relationship management, and DTO conversion.

## Exports

Methods on the `Store` struct:

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `CreateGlossaryEntry` | method | Full create with slug generation, duplicate term check, reciprocal adds, FTS5 index |
| `GetGlossaryEntry` | method | Query by ID, convert to DTO |
| `ListGlossaryEntries` | method | Filtered query with post-query tag matching, pagination, sorting |
| `UpdateGlossaryEntry` | method | Partial update with reciprocal diffs, index text rebuild, FTS5 reindex |
| `DeleteGlossaryEntry` | method | Cleanup reciprocal relationships, delete entry, remove from FTS5 |
| `UpdateGlossaryEntryCounts` | method | Set absolute usage/doc counts (called by TermScanHandler after scan) |
| `DecrementGlossaryEntryCounts` | method | Read-then-write decrement with floor at 0 (called on content deletion) |

## Internal Helpers

| Name | Description |
| ---- | ----------- |
| `getOrCreateGlossary` | Lazy `Glossary` entity creation -- queries the project with its glossary edge, creates one if missing |
| `generateSlug` | Lowercase, spaces to hyphens, strip non-alphanumeric, collapse consecutive hyphens |
| `ensureUniqueSlug` | Appends `-2`, `-3`, etc. if the base slug already exists within the glossary |
| `buildIndexText` | Joins term + shortDefinition + aliases + tags into a single lowercased string for FTS5 |
| `glossaryEntryToDTO` | Converts Ent entity to `repository.GlossaryEntryDTO` with nil-safe slices (empty array, not null) |
| `applyReciprocalAddsAfterCreate` | Adds source entry ID to each target's inverse relationship array |
| `applyReciprocalDiffsAfterUpdate` | Diffs old vs new relationship arrays, adds/removes source ID on target entries |
| `cleanupRelationshipReferences` | Removes deleted entry ID from all entries' relationship arrays across the entire glossary |
| `getRelationshipPairs` | Returns 6 source-to-inverse mapping pairs for reciprocal synchronization |
| `stripSelfReferences` | Removes an entry's own ID from its relationship arrays after save |
| `deduplicateSlice` | Removes duplicate strings from a slice using a seen-set |
| `addToStringSlice` | Appends an ID to a slice if not already present |
| `removeFromStringSlice` | Removes all occurrences of an ID from a slice |
| `containsString` | Checks if a string slice contains a specific value |
| `indexGlossaryForSearch` | Sends entry fields to FTS5 index; no-op if search provider is nil |
| `removeGlossaryFromSearch` | Removes entry from FTS5 index; no-op if search provider is nil |
| `hasAnyTag` | Case-insensitive any-match tag filter for post-query filtering |

## Reciprocal Relationship System

When a glossary entry references another entry in a relationship array, the inverse relationship is automatically maintained on the target entry.

| Source Field | Target Inverse Field | Type |
| ------------ | -------------------- | ---- |
| `broaderTerms` | `narrowerTerms` | Asymmetric |
| `narrowerTerms` | `broaderTerms` | Asymmetric |
| `relatedTerms` | `relatedTerms` | Symmetric |
| `synonyms` | `synonyms` | Symmetric |
| `antonyms` | `antonyms` | Symmetric |
| `seeAlso` | `seeAlso` | Symmetric |

Self-references are silently stripped after save. Duplicates are collapsed before persistence.

## Create Flow

1. Open per-project SQLite database via `dbPathFor(projectSlug)`
2. Get or create the `Glossary` parent entity via `getOrCreateGlossary()`
3. Check for duplicate term (case-insensitive) using `TermEqualFold`
4. Generate slug from term via `generateSlug()`, ensure uniqueness via `ensureUniqueSlug()`
5. Build composite `indexText` from term, shortDefinition, aliases, tags
6. Build Ent create operation with all provided fields
7. Deduplicate relationship arrays before save
8. Save the entry to the database
9. Strip self-references from relationship arrays (re-save; entry ID was unknown before first save)
10. Apply reciprocal adds on all target entries via `applyReciprocalAddsAfterCreate()`
11. Convert to DTO via `glossaryEntryToDTO()`
12. Index for FTS5 search via `indexGlossaryForSearch()`
13. Return the DTO

## Update Flow

1. Open per-project SQLite database
2. Query existing entry by ID (404 if not found)
3. Snapshot old entry for reciprocal relationship diffing
4. Build Ent update operation with only non-nil fields
5. Strip self-references and deduplicate relationship arrays
6. Rebuild `indexText` if any searchable field changed (term, shortDefinition, aliases, tags)
7. Save the updated entry
8. Apply reciprocal diffs on target entries via `applyReciprocalDiffsAfterUpdate()` -- adds source ID to newly referenced targets, removes from de-referenced targets
9. Convert to DTO
10. Re-index in FTS5 (delete + insert pattern)
11. Return the DTO

## Delete Flow

1. Open per-project SQLite database
2. Clean up reciprocal references via `cleanupRelationshipReferences()` -- queries all entries, removes deleted ID from every relationship array
3. Delete the entry by ID (404 if not found)
4. Remove from FTS5 index via `removeGlossaryFromSearch()`

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `ent` | `backend/internal/ent` | Ent ORM client and entity types |
| `glossary` | `backend/internal/ent/glossary` | Glossary entity predicates |
| `glossaryentry` | `backend/internal/ent/glossaryentry` | GlossaryEntry entity predicates and field constants |
| `entproject` | `backend/internal/ent/project` | Project entity predicates (aliased to avoid collision) |
| `repository` | `backend/internal/repository` | DTO types and input/output structs |

## Side Effects

- Reads and writes to the per-project SQLite database file at `storagePath/projectSlug.db`
- Modifies other glossary entries during reciprocal relationship synchronization (create, update, delete)
- Indexes and removes entries from the FTS5 search index when the search provider is available

## Notes

> [!WARNING] Tag filtering is performed post-query in Go code due to SQLite's lack of efficient JSON array-contains predicates. This means the `total` count returned by `ListGlossaryEntries` may not reflect tag-filtered results accurately -- it counts all entries matching the non-tag filters, while the returned `entries` slice is further filtered by tags. For glossaries with thousands of entries, this O(n) scan runs on every paginated page.
