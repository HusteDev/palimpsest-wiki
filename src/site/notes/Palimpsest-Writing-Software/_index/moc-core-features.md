---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/index/moc-core-features/","title":"MOC: Core Features","tags":["moc","features","core"],"updated":"2026-03-07T17:44:28.833-07:00"}
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

- [[Palimpsest-Writing-Software/features/glossary/diagrams/create-entry-flow\|Create Entry: Full Flow]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/read-entries-flow\|Read Entries: List and Get]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/update-entry-flow\|Update Entry: Reciprocal Diffs]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/delete-entry-flow\|Delete Entry: Cleanup]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/term-scan-flow\|Term Scan: Background Job]]
- [[Palimpsest-Writing-Software/features/glossary/diagrams/editor-integration-flow\|Editor Integration: Tooltip and Click]]

### Search

- [[Palimpsest-Writing-Software/features/search/diagrams/search-query-flow\|Search Query: Full Lifecycle]]
- [[Palimpsest-Writing-Software/features/search/diagrams/indexing-content-flow\|Indexing: Content Create/Update and Delete]]
- [[Palimpsest-Writing-Software/features/search/diagrams/indexing-glossary-flow\|Indexing: Glossary Create/Update and Delete]]
- [[Palimpsest-Writing-Software/features/search/diagrams/indexing-reindex-flow\|Indexing: Full Reindex Job]]
- [[Palimpsest-Writing-Software/features/search/diagrams/scope-resolution-flow\|Scope Resolution: Path Parsing and Tree Walking]]
- [[Palimpsest-Writing-Software/features/search/diagrams/editor-highlight-flow\|Editor Highlight: ProseMirror Decorations]]

### Panel System

- [[Palimpsest-Writing-Software/features/panel-system/diagrams/panel-rendering-flow\|Panel Rendering: Registry to Screen]]
- [[Palimpsest-Writing-Software/features/panel-system/diagrams/pop-out-dock-flow\|Pop-Out and Dock: Floating Panel Lifecycle]]

### ProseMirror Editor

- [[Palimpsest-Writing-Software/features/editor/diagrams/editor-init-flow\|Editor Initialization: Mount to Ready]]
- [[Palimpsest-Writing-Software/features/editor/diagrams/content-save-flow\|Content Editing: Change to Persist]]
- [[Palimpsest-Writing-Software/features/editor/diagrams/context-menu-flow\|Context Menu: Right-Click to Action]]

### Content Management

- [[Palimpsest-Writing-Software/features/content-management/diagrams/create-content-flow\|Create Content: Full Lifecycle]]
- [[Palimpsest-Writing-Software/features/content-management/diagrams/save-content-flow\|Save Content: Three-Tier Persistence]]
- [[Palimpsest-Writing-Software/features/content-management/diagrams/delete-content-flow\|Delete Content: Cleanup Chain]]

### Logging System

- [[Palimpsest-Writing-Software/architecture/logging/diagrams/logging-init-flow\|Logging Initialization: Two-Phase Startup]]
- [[Palimpsest-Writing-Software/architecture/logging/diagrams/log-output-flow\|Log Output: Cross-Layer Routing]]

### Plugin System

- [[Palimpsest-Writing-Software/features/plugins/diagrams/plugin-discovery-flow\|Plugin Discovery: Backend Scan to Frontend Load]]
- [[Palimpsest-Writing-Software/features/plugins/diagrams/content-rendering-flow\|Content Rendering: Plugin Renderer Lifecycle]]

### Settings System

- [[Palimpsest-Writing-Software/architecture/settings/diagrams/settings-load-flow\|Settings Load: Startup Initialization]]
- [[Palimpsest-Writing-Software/architecture/settings/diagrams/settings-update-flow\|Settings Update: Frontend Save to Disk]]

### Jobs System

- [[Palimpsest-Writing-Software/architecture/jobs/diagrams/job-submit-execute-flow\|Job Submit and Execute: Full Lifecycle]]
- [[Palimpsest-Writing-Software/architecture/jobs/diagrams/job-handler-dispatch-flow\|Job Handler Dispatch: Registration to Execution]]

## Related

- [[Palimpsest-Writing-Software/_index/moc-architecture\|MOC: Architecture & Infrastructure]]
- [[Palimpsest-Writing-Software/_index/moc-data-models\|MOC: Data Models]]
