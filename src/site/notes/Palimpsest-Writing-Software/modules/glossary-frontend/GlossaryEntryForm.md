---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-frontend/glossary-entry-form/","title":"GlossaryEntryForm.svelte","tags":["file","glossary-frontend"],"updated":"2026-03-05T05:32:43.049-07:00"}
---


# GlossaryEntryForm.svelte

> [!NOTE] Create/edit form for glossary entries with 8 collapsible sections covering all 30+ entry fields. Handles validation, save, and post-save event dispatch.

**Source:** `src/lib/components/glossary/GlossaryEntryForm.svelte`

## Exports

| Export | Kind | Description |
| ------ | ---- | ----------- |
| GlossaryEntryForm | default Svelte component | Create/edit form with 8 collapsible sections |

## Props

| Prop | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| mode | `'create'` \| `'edit'` | yes | Form behavior mode — determines save action and field initialization |
| entry | `GlossaryEntry` \| `null` | no | Entry to edit (edit mode); null for create mode |
| projectSlug | `string` | yes | Current project slug for API calls |
| initialTerm | `string` | no | Pre-populated term value (from context menu "Add to Glossary") |

## 8 Collapsible Sections

Each section is a collapsible accordion. Sections 1 and 2 are open by default; the rest are collapsed.

### 1. Basic (open by default)

- **term** — The glossary term itself (required, non-empty)
- **shortDefinition** — Brief definition displayed in tooltips (required, non-empty)
- **fullDefinition** — Extended definition displayed in detail view
- **status** — Entry status: draft, active, deprecated, archived

### 2. Categorization (open by default)

- **category** — Free-text category label
- **tags** — Array of tag strings, entered as comma-separated values
- **aliases** — Alternative names/spellings for the term, entered as comma-separated values

### 3. Linguistic

- **partOfSpeech** — Grammatical classification (noun, verb, adjective, etc.)
- **pronunciationGuide** — Phonetic or descriptive pronunciation

### 4. Scope

- **termScope** — Scope type: "full_project" or "targeted"
- **scopeTargets** — Array of content IDs (via ScopeTreePicker); empty when termScope is "full_project"
- **worldScope** — Free-text scope within the story world
- **spoilerLevel** — Numeric spoiler level (0 = no spoiler)

### 5. Examples

- **examples** — Example sentences demonstrating correct usage
- **nonExamples** — Sentences demonstrating incorrect usage or common mistakes
- **usageNotes** — Additional guidance on term usage
- **etymology** — Origin and history of the term

### 6. Visual

- **icon** — Icon identifier for display in lists
- **imageUrl** — URL to an associated image

### 7. Marking

- **autoMark** — Whether the term should be automatically marked in content
- **markColor** — CSS color for the dotted underline mark
- **markHoverColor** — CSS color for the hover background on marks
- **enableTooltip** — Whether hovering a mark shows the tooltip
- **enableHyperlink** — Whether clicking a mark navigates to the entry

### 8. Relationships

Each relationship field uses a [[Palimpsest-Writing-Software/modules/glossary-frontend/overview\|GlossaryTermPicker]] sub-component:

- **broaderTerms** — Parent/hypernym terms
- **narrowerTerms** — Child/hyponym terms
- **relatedTerms** — Associatively related terms
- **synonyms** — Terms with the same meaning
- **antonyms** — Terms with opposite meaning
- **seeAlso** — Cross-reference terms

## Save Flow

1. **Validate:** `term` must be non-empty, `shortDefinition` must be non-empty. If validation fails, display inline error messages and prevent submission.
2. **Build input DTO:** In edit mode, build a partial DTO with only changed fields. In create mode, include all fields.
3. **Call store method:** `glossaryStore.createEntry()` for create mode, `glossaryStore.updateEntry()` for edit mode.
4. **Dispatch event:** Emit `glossary:entry-saved` event via moduleEventBus with the entry ID.
5. **Spellcheck integration:** Auto-add invented terms to the spellcheck dictionary after a 2-second delay via `dictionaryStore.addWordIfMisspelled()` for the term and all aliases.

## Sub-components

| Component | Count | Purpose |
| --------- | ----- | ------- |
| GlossaryTermPicker | x6 | Search-and-select for each relationship field |
| ScopeTreePicker | x1 | Content scope selection tree for the Scope section |

## Dependencies

| Dependency | Type | Purpose |
| ---------- | ---- | ------- |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-store\|glossaryStore]] | internal | Create/update entry methods |
| moduleEventBus | internal | Dispatch `glossary:entry-saved` event |
| dictionaryStore | internal | Add invented terms to spellcheck dictionary |
| GlossaryTermPicker | child component | Relationship field search-and-select |
| ScopeTreePicker | child component | Content scope tree selection |

## Side Effects

- Dispatches `glossary:entry-saved` event on moduleEventBus after successful save.
- Calls `dictionaryStore.addWordIfMisspelled()` for the term and all aliases (2-second delay, retry with exponential backoff).
- Triggers network I/O via glossaryStore create/update methods.
