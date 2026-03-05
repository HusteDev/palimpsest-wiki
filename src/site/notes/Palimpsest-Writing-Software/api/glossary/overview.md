---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/overview/","title":"Glossary API Service","tags":["api","glossary"]}
---


# Glossary API Service

> [!NOTE]
> REST API service for glossary CRUD operations, served at `/api/projects/{slug}/glossary`.

## Base URL

```
/api/projects/{slug}/glossary
```

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler]]

## Authentication

None (local desktop application).

## Response Format

JSON with camelCase keys.

## Routes

| Method | Path | Description |
| ------ | ---- | ----------- |
| GET | `/glossary` | [[Palimpsest-Writing-Software/api/glossary/GET-list\|List Entries]] |
| POST | `/glossary` | [[Palimpsest-Writing-Software/api/glossary/POST-create\|Create Entry]] |
| GET | `/glossary/{id}` | [[Palimpsest-Writing-Software/api/glossary/GET-entry\|Get Entry]] |
| PUT | `/glossary/{id}` | [[Palimpsest-Writing-Software/api/glossary/PUT-update\|Update Entry]] |
| DELETE | `/glossary/{id}` | [[Palimpsest-Writing-Software/api/glossary/DELETE-entry\|Delete Entry]] |
| GET | `/glossary/{id}/occurrences` | [[Palimpsest-Writing-Software/api/glossary/GET-occurrences\|Get Occurrences]] |

## Common Error Responses

| Status | Description |
| ------ | ----------- |
| 404 | Project or entry not found |
| 405 | Method not allowed |
| 500 | Server error |

## Event Schemas

Three glossary-specific events flow through the `moduleEventBus`:

| Event | Description |
| ----- | ----------- |
| [[Palimpsest-Writing-Software/api/glossary/event-glossary-view-term\|glossary:view-term]] | User clicks a glossary mark in the editor |
| [[Palimpsest-Writing-Software/api/glossary/event-glossary-entry-saved\|glossary:entry-saved]] | Entry created or updated |
| [[Palimpsest-Writing-Software/api/glossary/event-glossary-entry-deleted\|glossary:entry-deleted]] | Entry deleted |

## Data Model

[[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|Response DTO]]
