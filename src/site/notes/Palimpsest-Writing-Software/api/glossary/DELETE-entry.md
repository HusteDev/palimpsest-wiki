---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/delete-entry/","title":"DELETE /glossary/{id}","tags":["api","glossary"]}
---


# DELETE /glossary/{id}

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler.deleteEntry()]]

> [!NOTE]
> Deletes a glossary entry. Cleans up reciprocal relationships and triggers mark removal from content.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |
| `id` | string | Entry UUID |

## Request Body

None.

## Success Response

**Status:** `204 No Content`

## Error Responses

| Status | Description |
| ------ | ----------- |
| 404 | Entry not found |
| 500 | Server error |

> [!WARNING]
> Deletion is permanent. All reciprocal relationship references are cleaned from other entries. A background `term_scan` 'remove' job strips all glossaryLink marks for this term from content documents.
