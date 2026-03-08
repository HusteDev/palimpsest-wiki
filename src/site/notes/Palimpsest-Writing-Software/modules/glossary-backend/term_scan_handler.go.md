---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-backend/term-scan-handler-go/","title":"term_scan_handler.go","tags":["file","glossary-backend"],"updated":"2026-03-05T05:32:42.715-07:00"}
---


# `term_scan_handler.go`

**Module:** [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Glossary Backend]]
**Path:** `backend/internal/glossary/term_scan_handler.go`

> [!NOTE] Background job handler that applies or removes `glossaryLink` ProseMirror marks in content documents. Decoupled from the repository via callback functions.

## TermScanHandler Struct

```go
type TermScanHandler struct {
    entryLoader   EntryLoader      // Loads glossary entry data
    contentLoader ContentLoader    // Loads all content for a project
    bodySaver     ContentBodySaver // Saves updated content body
    countUpdater  CountUpdater     // Updates glossary entry usage counts
    log           *logging.Logger  // Injected logger
}
```

## Callback Types

### TermData

```go
type TermData struct {
    ID              string   // Glossary entry UUID
    Term            string   // The glossary term (e.g., "Dragon")
    Slug            string   // URL-safe slug
    Aliases         []string // Alternative names / abbreviations
    AutoMark        bool     // Whether auto-marking is enabled
    MarkColor       string   // CSS color for the mark highlight
    MarkHoverColor  string   // CSS color for the mark hover highlight
    ScopeTargets    []string // Slug paths this term is scoped to (empty = all)
    EnableHyperlink bool     // Whether clicking the mark navigates to the entry
}
```

### ContentData

```go
type ContentData struct {
    ID       string         // Content document UUID
    SlugPath []string       // Slug path segments (e.g., ["series-1", "book-1", "chapter-3"])
    Body     map[string]any // ProseMirror JSON document (may be nil/empty for non-leaf content)
}
```

### Callback Signatures

| Type | Signature | Description |
| ---- | --------- | ----------- |
| `EntryLoader` | `func(ctx, projectSlug, entryID) (*TermData, error)` | Loads a glossary entry by ID; returns nil if already deleted |
| `ContentLoader` | `func(ctx, projectSlug) ([]ContentData, error)` | Loads all content items for a project as a flat list |
| `ContentBodySaver` | `func(ctx, projectSlug, contentID, body) error` | Saves an updated ProseMirror JSON body for a content item |
| `CountUpdater` | `func(ctx, projectSlug, entryID, usageCount, docCount) error` | Updates usage/doc counts on a glossary entry after scanning |

## Job Registration

- **Job type:** `"term_scan"`
- **Registered in:** `backend/cmd/server/main.go` via `jobManager.RegisterHandler(termScanHandler)`

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `NewTermScanHandler` | constructor | Takes 4 callbacks (`EntryLoader`, `ContentLoader`, `ContentBodySaver`, `CountUpdater`) and returns a configured handler. Logger is auto-initialized with component `"term-scan"`. |
| `JobType()` | method | Returns `"term_scan"` -- must match the job type used when submitting term scan jobs |
| `Execute(ctx, job)` | method | Dispatches to `executeScan` or `executeRemove` based on `job.Params["action"]` (defaults to `"scan"`) |

## executeScan Flow

1. Load the glossary entry via `entryLoader(ctx, projectSlug, entryID)`
2. Load all content items for the project via `contentLoader(ctx, projectSlug)`
3. If `autoMark` is `false`, delegate to `removeFromAllContent()` to strip existing marks and return
4. Build a `prosemirror.PatternTarget` from term + aliases
5. Build a `prosemirror.MarkConfig` with `"glossaryLink"` mark type and attribute builder (sets `termId`, `termSlug`, optional `color`, `hoverColor`, `enableHyperlink`)
6. For each content item:
   - Skip if body is empty (non-leaf content)
   - Check cancellation via `job.Progress.IsCancelled()`
   - Check scope via `scope.ItemAppliesToContent(scopeTargets, slugPath)` -- if out of scope, remove any existing marks for this term and save if changed
   - If in scope: remove existing marks for this term (clean slate), extract text segments, find pattern matches, apply `glossaryLink` marks to matched positions
   - Save the document if `docChanged()` detects differences
   - Accumulate mark count and document count
7. Update stored counts on the glossary entry via `countUpdater()`
8. Log completion with processed count, total marks, and docs-with-marks count

## executeRemove Flow

1. Load all content items for the project via `contentLoader(ctx, projectSlug)`
2. Delegate to `removeFromAllContent()` with the deleted entry ID
3. Reset usage and doc counts to zero via `countUpdater()`

## removeFromAllContent Helper

Iterates all content items and removes `glossaryLink` marks matching the target entry ID:

1. For each content item with a non-empty body:
   - Check cancellation
   - Call `prosemirror.RemoveMarksForTarget(body, "glossaryLink", termID, "termId")`
   - Save if `docChanged()` detects differences
   - Report progress via `job.Progress.Update()`
2. Log completion with processed and modified counts

## docChanged Helper

Compares two ProseMirror documents by JSON serialization (`json.Marshal` both, compare resulting strings). Returns `true` if the documents differ. If serialization fails for either document, assumes changed to be safe (avoids skipping a necessary save).

## Wiring in main.go

The 4 callbacks are constructed as closures in `backend/cmd/server/main.go` (lines 159-206):

| Callback | Wiring |
| -------- | ------ |
| `EntryLoader` | Calls `store.GetGlossaryEntry()`, maps `GlossaryEntryDTO` fields to `TermData` struct |
| `ContentLoader` | Calls `store.ListAllContent()`, maps each `ContentDTO` to `ContentData` (ID, SlugPath, Body) |
| `ContentBodySaver` | Calls `store.UpdateContentBody()` directly |
| `CountUpdater` | Type-asserts `store` to `*sqlite.Store`, calls `UpdateGlossaryEntryCounts()`; no-op if store is not SQLite |

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `jobs` | `backend/internal/jobs` | `JobContext` struct and `JobHandler` interface |
| `logging` | `backend/internal/logging` | Structured logger with `"term-scan"` component |
| `prosemirror` | `backend/internal/prosemirror` | Mark manipulation: `RemoveMarksForTarget`, `ExtractTextSegments`, `FindPatternMatches`, `ApplyMarks` |
| `scope` | `backend/internal/scope` | `ItemAppliesToContent` for visibility filtering |
| `encoding/json` | stdlib | JSON serialization for `docChanged()` comparison |
| `fmt` | stdlib | Error wrapping |
| `context` | stdlib | Context for cancellation/timeout |

## Side Effects

- **Reads and writes content document bodies** -- loads all content via `contentLoader`, saves modified documents via `bodySaver`. Documents may be modified even if the triggering entry was not directly related (scope changes can cause mark removal).
- **Updates glossary entry usage/doc counts** -- sets absolute counts after scan, resets to zero after remove. Uses `countUpdater` callback.
- **Uses ProseMirror package** for mark manipulation (`RemoveMarksForTarget`, `ExtractTextSegments`, `FindPatternMatches`, `ApplyMarks`).
- **Uses scope package** for content visibility filtering (`ItemAppliesToContent`).

## Notes

> [!WARNING] The `executeScan` method uses a clean-slate approach: it removes all existing marks for the term before re-scanning. This means every scan is a full re-application, not an incremental update. For projects with many content documents, this ensures correctness at the cost of processing time.
