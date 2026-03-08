---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/index/moc-core-features/","title":"MOC: Core Features","tags":["moc","features","core"],"updated":"2026-03-07T18:40:55.257-07:00"}
---


# Core Features — Map of Content

> [!NOTE] Features included in the free Core license, targeting authors using the locally installed desktop application.

## Features

- [[Palimpsest-Writing-Software/features/glossary/overview\|Glossary System]]
- [[Palimpsest-Writing-Software/features/search/overview\|Search Engine]]
- [[Palimpsest-Writing-Software/features/panel-system/overview\|Panel System]]
- [[Palimpsest-Writing-Software/features/editor/overview\|ProseMirror Editor]]
- [[Palimpsest-Writing-Software/features/content-management/overview\|Content Management]]
- [[Palimpsest-Writing-Software/features/plugins/overview\|Plugin System]]
- [[Palimpsest-Writing-Software/features/typography/font-manager\|Font Manager]]

## Architecture / Infrastructure

- [[Palimpsest-Writing-Software/architecture/logging/overview\|Logging System]]
- [[Palimpsest-Writing-Software/architecture/settings/overview\|Settings System]]
- [[Palimpsest-Writing-Software/architecture/jobs/overview\|Jobs System]]

## Modules

- [[Palimpsest-Writing-Software/modules/glossary-backend/overview\|Glossary Backend]]
- [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|Glossary Frontend]]
- [[Palimpsest-Writing-Software/modules/search-backend/overview\|Search Backend]]
- [[Palimpsest-Writing-Software/modules/search-frontend/overview\|Search Frontend]]
- [[Palimpsest-Writing-Software/modules/panel-frontend/overview\|Panel Frontend]]
- [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
- [[Palimpsest-Writing-Software/modules/content-frontend/overview\|Content Frontend]]
- [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend]]
- [[Palimpsest-Writing-Software/modules/logging-backend/overview\|Logging Backend]]
- [[Palimpsest-Writing-Software/modules/logging-frontend/overview\|Logging Frontend]]
- [[Palimpsest-Writing-Software/modules/plugins-backend/overview\|Plugins Backend]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/overview\|Plugins Frontend]]
- [[Palimpsest-Writing-Software/modules/settings-backend/overview\|Settings Backend]]
- [[Palimpsest-Writing-Software/modules/settings-frontend/overview\|Settings Frontend]]
- [[Palimpsest-Writing-Software/modules/jobs-backend/overview\|Jobs Backend]]

## APIs

- [[Palimpsest-Writing-Software/api/glossary/overview\|Glossary API Service]]
- [[Palimpsest-Writing-Software/api/search/overview\|Search API Service]]
- [[Palimpsest-Writing-Software/api/content/overview\|Content API Service]]
- [[Palimpsest-Writing-Software/api/plugins/overview\|Plugin API Service]]
- [[Palimpsest-Writing-Software/api/settings/overview\|Settings API Service]]
- [[Palimpsest-Writing-Software/api/jobs/overview\|Jobs API Service]]

## Data Models

- [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|GlossaryEntry (Ent Schema)]]
- [[Palimpsest-Writing-Software/data-models/Glossary\|Glossary (Parent Entity)]]
- [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|GlossaryEntryDTO (Transfer Objects)]]
- [[Palimpsest-Writing-Software/data-models/SearchResult\|SearchResult (Search DTOs)]]
- [[Palimpsest-Writing-Software/data-models/PanelLayout\|PanelLayout (Panel State)]]
- [[Palimpsest-Writing-Software/data-models/Content\|Content (Ent Schema)]]
- [[Palimpsest-Writing-Software/data-models/ContentType\|ContentType (Hierarchy Levels)]]

## Event Schemas

- [[Palimpsest-Writing-Software/api/glossary/event-glossary-view-term\|glossary:view-term]]
- [[Palimpsest-Writing-Software/api/glossary/event-glossary-entry-saved\|glossary:entry-saved]]
- [[Palimpsest-Writing-Software/api/glossary/event-glossary-entry-deleted\|glossary:entry-deleted]]

## Canvases / Diagrams

### Glossary


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/create-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Create Entry: Full Flow

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/read-entries-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Read Entries: List and Get

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/update-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Update Entry: Reciprocal Diffs

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/delete-entry-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Delete Entry: Cleanup

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/term-scan-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Term Scan: Background Job

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/glossary/diagrams/editor-integration-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Editor Integration: Tooltip and Click

</div>





# Excalidraw Data

## Text Elements




</div></div>


### Search


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/search/diagrams/search-query-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Search Query: Full Lifecycle

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/search/diagrams/indexing-content-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Indexing: Content Create/Update and Delete

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/search/diagrams/indexing-glossary-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Indexing: Glossary Create/Update and Delete

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/search/diagrams/indexing-reindex-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Indexing: Full Reindex Job

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/search/diagrams/scope-resolution-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Scope Resolution: Path Parsing and Tree Walking

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/search/diagrams/editor-highlight-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Editor Highlight: ProseMirror Decorations

</div>





# Excalidraw Data

## Text Elements




</div></div>


### Panel System


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/panel-system/diagrams/panel-rendering-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Panel Rendering: Registry to Screen

</div>





# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/features/panel-system/diagrams/pop-out-dock-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Pop-Out and Dock: Floating Panel Lifecycle

</div>




# Excalidraw Data

## Text Elements




</div></div>


### ProseMirror Editor


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


### Content Management


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Create Content: Full Lifecycle

</div>




# Excalidraw Data

## Text Elements

Title: Create Content Full Lifecycle Subtitle: ContentTree -> CreateContentModal -> API -> Repository -> DB 
Zone 1 - Frontend UI:
- Click "Add" in ContentTree
- CreateContentModal opens (3-step wizard)
- Step 1: Select content type (filtered by parent level)
- Step 2: Select parent (if level > 1)
- Step 3: Enter title (or auto-title checkbox)
- contentApi.createContent(projectSlug, parentSlug, input)

Zone 2 - API Layer:
- HTTP POST /api/projects/{slug}/content or /{parent}/children
- Validate: title required, contentTypeId required
- Call store.CreateContent()
- Return ContentResponse JSON

Zone 3 - Repository / Database:
- Get project by slug
- Get content type by ID
- Resolve parent (if any)
- Slugify(title) -> slug
- nextSiblingOrder() -> order
- Create Ent entity with all fields
- buildSlugPath() (walk parent chain)
- FTS5 Index (search indexing)




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Save Content: Three-Tier Persistence

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Delete Content: Cleanup Chain

</div>




# Excalidraw Data

## Text Elements
1. User clicks Delete in ContentTree 
Confirmation Dialog 
2. contentApi.deleteContent (projectSlug, slugPath) 
3. HTTP DELETE /api/projects/{slug}/content/{path} 
4. Load content + ProseMirror JSON 
5. ExtractMarksOfType() glossaryLink marks 
6. Group by termId count occurrences 
7. Decrement glossary counts (usageCount, docCount) best effort 
8. store.DeleteContent() 
9. Resolve content by slug path 
10. Capture parent ID for post-delete ops 
11. Delete via Ent (cascade handles children) 
12. Remove from FTS5 search index 
13. RecalculateWordCount (recursive up tree) 
14. ReorderAndRenameSiblings (two-pass: temp slugs -> final slugs) 
15. Return 204 No Content 
Frontend (Blue) 
Handler (Glossary Cleanup) 
Repository (Delete + Cascade Cleanup) 
Cleanup Chain: FTS5 -> Word Count -> Sibling Reorder 




</div></div>


### Logging System


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Logging Initialization: Two-Phase Startup

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Log Output: Cross-Layer Routing

</div>




# Excalidraw Data

## Text Elements




</div></div>


### Plugin System


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Plugin Discovery: Backend Scan to Frontend Load

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Content Rendering: Plugin Renderer Lifecycle

</div>




# Excalidraw Data

## Text Elements




</div></div>


### Settings System


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Settings Load: Startup Initialization

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">

<div class="markdown-embed-title">

# Settings Update: Frontend Save to Disk

</div>




# Excalidraw Data

## Text Elements




</div></div>


### Jobs System


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/architecture/jobs/diagrams/job-submit-execute-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Job Submit and Execute: Full Lifecycle

</div>




# Excalidraw Data

## Text Elements




</div></div>


<div class="transclusion internal-embed is-loaded"><a class="markdown-embed-link" href="/palimpsest-writing-software/architecture/jobs/diagrams/job-handler-dispatch-flow/" aria-label="Open link"><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="svg-icon lucide-link"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></a><div class="markdown-embed">

<div class="markdown-embed-title">

# Job Handler Dispatch: Registration to Execution

</div>




# Excalidraw Data

## Text Elements




</div></div>


## Related

- [[Palimpsest-Writing-Software/_index/moc-architecture\|MOC: Architecture & Infrastructure]]
- [[Palimpsest-Writing-Software/_index/moc-data-models\|MOC: Data Models]]
