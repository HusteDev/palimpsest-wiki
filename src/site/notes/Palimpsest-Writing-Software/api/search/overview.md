---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/search/overview/","title":"Search API Service","tags":["api","search"],"updated":"2026-03-05T05:32:38.376-07:00"}
---


# Search API Service

> [!NOTE]
> REST API service for unified full-text search across content and glossary entries, served at `/api/projects/{slug}/search`.

## Base URL

```
/api/projects/{slug}/search
```

## Handler

[[Palimpsest-Writing-Software/modules/search-backend/handlers-search.go\|SearchHandler]]

## Authentication

None (local desktop application).

## Response Format

JSON with camelCase keys.

## Routes

| Method | Path | Description |
| ------ | ---- | ----------- |
| GET | `/search` | [[Palimpsest-Writing-Software/api/search/GET-search\|Search]] |

## Common Error Responses

| Status | Description |
| ------ | ----------- |
| 400 | Empty or missing search query |
| 404 | Project not found |
| 405 | Method not allowed |

## Data Model

[[Palimpsest-Writing-Software/data-models/SearchResult\|Response DTO]]
