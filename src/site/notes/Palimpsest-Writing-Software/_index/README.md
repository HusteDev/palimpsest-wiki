---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/index/readme/","title":"Palimpsest Technical Wiki","tags":["moc","index","gardenEntry"]}
---


# Palimpsest Technical Wiki

> [!NOTE] Living technical documentation for the Palimpsest writing software — a unified platform for every stakeholder in the publishing pipeline.

This vault documents **implemented features and systems only**. Planning, roadmap, and future feature tracking live elsewhere. If a page exists here, the code exists in the codebase.

## Tech Stack

- **Backend:** Go (sidecar server via Tauri)
- **Frontend:** Svelte 5 (SvelteKit)
- **Desktop:** Tauri v2
- **API:** REST with DTO isolation
- **Database:** Per-project SQLite via Ent ORM
- **Editor:** ProseMirror

## Navigation

### Architecture

<!-- [[architecture/overview\|System Architecture Overview]] — not yet documented -->

### Maps of Content

- [[Palimpsest-Writing-Software/_index/moc-architecture\|Architecture & Infrastructure]]
- [[Palimpsest-Writing-Software/_index/moc-core-features\|Core Features (Free License)]]
- [[Palimpsest-Writing-Software/_index/moc-data-models\|Data Models & Schemas]]

## Repositories

- **Codebase:** https://github.com/HusteDev/palimpsest
- **This Wiki:** https://github.com/HusteDev/palimpsest-wiki

## Conventions

- Every page uses standardized frontmatter (see `_templates/`)
- Every reference to another documented entity uses a `[[wikilink]]`
- Every feature with dataflow has canvas diagrams in a `diagrams/` subfolder
- Status values: `draft` → `review` → `stable` → `deprecated`

---

_Last updated: 2026-03-04_
