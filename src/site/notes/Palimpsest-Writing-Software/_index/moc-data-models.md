---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/index/moc-data-models/","title":"MOC: Data Models & Schemas","tags":["moc","data-model"]}
---


# Data Models & Schemas — Map of Content

> [!NOTE] Database schemas, Ent ORM entities, DTOs, and TypeScript types used throughout the Palimpsest application.

## Data Models

### Glossary

- [[Palimpsest-Writing-Software/data-models/GlossaryEntry\|GlossaryEntry]] — 37-field Ent ORM schema for glossary terms
- [[Palimpsest-Writing-Software/data-models/Glossary\|Glossary]] — Parent entity (1:1 with Project)
- [[Palimpsest-Writing-Software/data-models/GlossaryEntryDTO\|GlossaryEntryDTO]] — Go DTOs and TypeScript types for API transfer

### Search

- [[Palimpsest-Writing-Software/data-models/SearchResult\|SearchResult]] — Go structs and TypeScript interfaces for search requests/responses

### Panel System

- [[Palimpsest-Writing-Software/data-models/PanelLayout\|PanelLayout]] — Panel layout state: docked configs, floating instances, UI state

## Related

- [[Palimpsest-Writing-Software/_index/moc-architecture\|MOC: Architecture & Infrastructure]]
- [[Palimpsest-Writing-Software/_index/moc-core-features\|MOC: Core Features]]
