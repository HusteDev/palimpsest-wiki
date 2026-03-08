---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/content-frontend/overview/","title":"Content Frontend Module","tags":["module","content-frontend"],"updated":"2026-03-05T06:20:29.471-07:00"}
---


# Content Frontend

> [!NOTE] TypeScript types, API client, save services, and utility modules for managing hierarchical content in the Svelte frontend. Provides the complete client-side stack from type definitions through three-tier persistence.

## Responsibilities

- Define TypeScript interfaces for Content, ContentType, and all API request/response shapes
- Provide typed HTTP client methods for content CRUD, tree operations, and reordering
- Manage three-tier persistence: localStorage drafts → backend sync → autosave orchestration
- Provide content scope filtering utilities for glossary term applicability
- Expose utility functions for content type display (labels, icons, colors)

## Public API Surface

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `Content` | interface | Content item with hierarchy, metadata, and ProseMirror JSON |
| `ContentType` | interface | Content type definition with level, metadata, and hierarchy info |
| `CreateContentInput` | interface | Request DTO for creating content |
| `UpdateContentInput` | interface | Request DTO for updating content (partial, all fields optional) |
| `ProseMirrorDocJSON` | type | Alias for serialized ProseMirror document root |
| `contentApi` | object | Grouped export of all content API functions |
| `APIClient` | class | Base HTTP client with typed methods and error handling |
| `APIError` | class | Error class with status, statusText, and convenience getters |
| `localDraftService` | singleton | LocalDraftService instance for localStorage draft management |
| `saveService` | singleton | SaveService instance for backend synchronization |
| `autosaveService` | singleton | AutosaveService instance for save trigger orchestration |
| `filterByScope` | function | Filter scoped items by content slug path |
| `isContentWithinTarget` | function | Check if content path is within a scope target |

## Internal Structure

The module is organized into four layers:

**Types** — `[[modules/content-frontend/contentTypes|contentTypes]]` defines all TypeScript interfaces shared between the API client, UI components, and save services.

**API Client** — `[[modules/content-frontend/contentApi|contentApi]]` wraps the base `[[modules/content-frontend/apiClient|APIClient]]` to provide typed content CRUD methods plus display helpers.

**Save Services** — Three coordinated singletons form the persistence stack:
- `[[modules/content-frontend/localDraftService|localDraftService]]` — localStorage layer with debounced saves
- `[[modules/content-frontend/saveService|saveService]]` — backend sync layer
- `[[modules/content-frontend/autosaveService|autosaveService]]` — trigger orchestration layer

**Utilities** — `[[modules/content-frontend/contentScopeFilter|contentScopeFilter]]` provides scope-matching for determining which items apply to a given content path.

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `$lib/config` | internal | `BACKEND_URL`, `DRAFT_SAVE_DELAY`, `LOCAL_STORAGE_PREFIX` |
| `$lib/services/loggerService` | internal | Structured logging (`'api'`, `'draft'`, `'save'`, `'autosave'`) |
| `$lib/services/PollingTimer` | internal | Debounce and polling timer for draft/autosave |
| `$lib/services/wordCountService` | internal | Reactive word count store updates |
| `$lib/stores/preferences` | internal | User preferences for autosave settings |
| `svelte/store` | external | `get()` for reading store values in services |

## Related

- [[Palimpsest-Writing-Software/features/content-management/overview\|Feature: Content Management]]
- [[Palimpsest-Writing-Software/modules/content-backend/overview\|Content Backend Module]]
- [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend Module]]
- [[Palimpsest-Writing-Software/data-models/Content\|Data Model: Content]]
- [[Palimpsest-Writing-Software/data-models/ContentType\|Data Model: ContentType]]
