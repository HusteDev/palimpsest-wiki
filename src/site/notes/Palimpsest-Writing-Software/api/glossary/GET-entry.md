---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/api/glossary/get-entry/","title":"GET /glossary/{id}","tags":["api","glossary"]}
---


# GET /glossary/{id}

## Handler

[[Palimpsest-Writing-Software/modules/glossary-backend/handlers-glossary.go\|GlossaryHandler.getEntry()]]

> [!NOTE]
> Retrieves a single glossary entry by ID.

## Authentication

None.

## Path Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| `slug` | string | Project slug |
| `id` | string | Entry UUID |

## Query Parameters

None.

## Request Body

None.

## Success Response

**Status:** `200 OK`

Returns the full [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|GlossaryEntry]] object.

## Error Responses

| Status | Description |
| ------ | ----------- |
| 404 | Entry not found |
| 500 | Server error |
