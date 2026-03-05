---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/schema/","title":"schema.ts","tags":["file","editor-frontend","prosemirror"]}
---


# `schema.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/schema.ts`

> [!NOTE] Defines the ProseMirror document schema — the set of legal node types and mark types that govern the structure and formatting of editor content.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `schema` | `Schema` instance | The fully composed schema used by all editor components |

## Schema Composition

The schema is built in layers using `OrderedMap`:

1. **Basic nodes** from `prosemirror-schema-basic` (`doc`, `paragraph`, `text`, `blockquote`, `heading`, `horizontal_rule`, `code_block`, `hard_break`)
2. **List nodes** added via `addListNodes()` from `prosemirror-schema-list` (`bullet_list`, `ordered_list`, `list_item`)
3. **Custom nodes** defined in this file (`details`, `summary`, `image`)
4. **Table nodes** from `prosemirror-tables` (`table`, `table_row`, `table_cell`, `table_header`) with a `background` cell attribute
5. **Basic marks** from `prosemirror-schema-basic` (`strong`, `em`, `code`, `link`)
6. **Custom marks** defined in this file (`strikethrough`, `underline`, `highlight`, `glossaryLink`)

## Custom Nodes

### `details`

| Property | Value |
| -------- | ----- |
| Content | `'summary block*'` |
| Group | `'block'` |
| ParseDOM | `<details>` |

Collapsible content container. Must contain exactly one `summary` node followed by zero or more block nodes.

### `summary`

| Property | Value |
| -------- | ----- |
| Content | `'inline*'` |
| Group | — (not a standalone block) |
| ParseDOM | `<summary>` |

Summary line displayed as the clickable header of a `details` block.

### `image`

| Property | Value |
| -------- | ----- |
| Content | — (leaf) |
| Group | `'inline'` |
| Inline | `true` |
| Draggable | `true` |
| Attrs | `src` (required), `alt` (default: null), `title` (default: null) |
| ParseDOM | `<img[src]>` |

Inline draggable image node. Overrides the basic schema's block-level image to allow inline placement.

## Custom Marks

### `strikethrough`

| Property | Value |
| -------- | ----- |
| Attrs | none |
| ParseDOM | `<s>`, `<del>`, `<strike>`, `text-decoration: line-through` |
| ToDOM | `['s', 0]` |

### `underline`

| Property | Value |
| -------- | ----- |
| Attrs | none |
| ParseDOM | `<u>`, `text-decoration: underline` |
| ToDOM | `['u', 0]` |

### `highlight`

| Property | Value |
| -------- | ----- |
| Attrs | `color` (default: null) |
| ParseDOM | `<mark>` with optional `data-color` attribute |
| ToDOM | `['mark', { 'data-color': color }, 0]` |

Persistent inline highlight. The optional `color` attribute allows themed or custom highlight colors.

### `glossaryLink`

| Property | Value |
| -------- | ----- |
| Attrs | `termId` (required), `termSlug` (default: `''`), `color` (default: `''`), `hoverColor` (default: `''`), `enableHyperlink` (default: `false`) |
| Inclusive | `false` |
| ParseDOM | `<span[data-glossary-term]>` |
| ToDOM | `['span', { class: 'glossary-mark', 'data-glossary-term': termId, ... }, 0]` |

Applied by the backend `TermScanHandler` to identify glossary terms in content. The `inclusive: false` setting prevents the mark from extending when users type at the edges of a marked term.

> [!IMPORTANT] The `glossaryLink` mark is the only custom mark that carries semantic data (`termId`) linking to an external entity. All other marks are purely formatting-based.

The mark renders with CSS classes:
- `.glossary-mark` — always present (dotted underline)
- `.glossary-mark-hyperlink` — added when `enableHyperlink: true` (pointer cursor)

Custom colors are applied via inline `style` attributes (`border-bottom-color`, `--glossary-mark-hover-bg`).

## Table Cell Attributes

The `tableNodes()` factory is configured with a `background` cell attribute:

```ts
cellAttributes: {
    background: {
        default: null,
        getFromDOM(dom) { return dom.style.backgroundColor || null; },
        setDOMAttr(value, attrs) { if (value) attrs.style += `background-color: ${value};`; }
    }
}
```

This allows individual table cells to have custom background colors while keeping the default `null` for unstyled cells.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Schema`, `Mark`, `MarkSpec`, `NodeSpec`, `Node` | `prosemirror-model` | Schema definition types |
| `nodes`, `marks` (as `basicNodes`, `basicMarks`) | `prosemirror-schema-basic` | Base node and mark specs |
| `addListNodes` | `prosemirror-schema-list` | List node composition |
| `tableNodes` | `prosemirror-tables` | Table node specs with cell attributes |
| `OrderedMap` | `orderedmap` | Deterministic node ordering |

## Side Effects

None. This file only exports the `schema` constant. No global state, no registrations, no environment reads.
