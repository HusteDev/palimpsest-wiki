---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/glossary-frontend/glossary-mark/","title":"glossaryMark.ts","tags":["file","glossary-frontend","prosemirror"],"updated":"2026-03-05T05:32:43.153-07:00"}
---


# glossaryMark.ts

> [!NOTE] ProseMirror plugin providing hover tooltips and click-to-navigate behavior for glossaryLink marks. Includes caching and event-driven cache invalidation.

**Source:** `src/lib/components/editor/proseMirror/plugins/glossaryMark.ts`

## Exports

| Export | Kind | Description |
| ------ | ---- | ----------- |
| createGlossaryMarkPlugin | function | Factory function — creates a ProseMirror plugin instance from a GlossaryMarkProvider |
| GlossaryMarkProvider | interface | Contract defining term info retrieval and navigation callbacks |

## GlossaryMarkProvider Interface

```typescript
interface GlossaryMarkProvider {
    getTermInfo(termId: string): Promise<{
        term: string;
        shortDefinition: string;
        enableHyperlink: boolean;
        enableTooltip: boolean;
    } | null>;
    navigateToTerm(termId: string, termSlug: string): void;
}
```

- **getTermInfo(termId)** — Fetches display information for a glossary term by its UUID. Returns null if the term is not found. Called when the user hovers over a glossary mark in the editor.
- **navigateToTerm(termId, termSlug)** — Navigates to the glossary entry detail view. Called when the user clicks a hyperlink-enabled glossary mark.

## TooltipManager (Internal)

The `TooltipManager` class is an internal implementation detail that manages the tooltip DOM element and its lifecycle:

- **DOM element:** Creates and manages a `div.glossary-tooltip` element appended to the editor container.
- **Cache:** Maintains a `Map<string, GlossaryTermInfo | null>` to avoid redundant API calls for term information.
- **show(termId, anchorElement):** Fetches term info (from cache or API), positions the tooltip below the glossary mark element, and displays the term name and short definition.
- **scheduleHide():** Starts a 200ms delay before hiding the tooltip. This delay allows the user to move their mouse from the mark to the tooltip itself without it disappearing.
- **cancelHide():** Cancels the scheduled hide timer. Called when the mouse enters the tooltip element, keeping it visible while the user reads it.
- **clearCache():** Empties the term info cache. Called in response to `glossary:entry-saved` and `glossary:entry-deleted` events to ensure stale data is not displayed.

## glossaryLink Mark Spec

Defined in the ProseMirror schema (`src/lib/components/editor/proseMirror/schema.ts`):

| Attribute | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| termId | `string` | required | Glossary entry UUID |
| termSlug | `string` | `''` | URL-safe slug for the term |
| color | `string` | `''` | CSS color for the dotted underline |
| hoverColor | `string` | `''` | CSS color for the hover background |
| enableHyperlink | `boolean` | `false` | Whether clicking the mark navigates to the entry |

## Rendered DOM

When a glossaryLink mark is rendered in the editor, it produces the following HTML structure:

```html
<span class="glossary-mark [glossary-mark-hyperlink]"
      data-glossary-term="{termId}" data-glossary-slug="{termSlug}"
      data-glossary-color="{color}" data-glossary-hover-color="{hoverColor}"
      data-enable-hyperlink="true|false"
      style="border-bottom-color: {color}; --glossary-mark-hover-bg: {hoverColor}">
```

- The `glossary-mark-hyperlink` class is conditionally added when `enableHyperlink` is true, enabling the pointer cursor.
- Data attributes store mark metadata for the plugin's event handlers to read.
- Inline styles set the underline color and a CSS custom property for the hover background.

## CSS

- **Default state:** 2px dotted underline using the theme accent color (or custom `color` attribute if set).
- **Hover state:** Solid underline with optional background color (from `hoverColor` attribute via the `--glossary-mark-hover-bg` CSS custom property).
- **Hyperlink-enabled:** Pointer cursor when `enableHyperlink` is true, indicating the mark is clickable.

## Provider Wiring

The provider is wired in `TextEditor.svelte` when the ProseMirror editor is initialized:

- **getTermInfo** — Calls `glossaryApi.getEntry()` to fetch the entry from the backend, then returns a struct with `term`, `shortDefinition`, `enableHyperlink`, and `enableTooltip` fields.
- **navigateToTerm** — Dispatches a `glossary:view-term` event via moduleEventBus with the entry ID, which is received by GlossaryPanel to switch to detail view.
- **Cache invalidation** — Subscribes to `glossary:entry-saved` and `glossary:entry-deleted` events on moduleEventBus. When either fires, calls `clearCache()` on the TooltipManager so the next hover fetches fresh data.

## Dependencies

| Dependency | Type | Purpose |
| ---------- | ---- | ------- |
| ProseMirror | external | Editor framework — Plugin, PluginKey, EditorView |
| [[Palimpsest-Writing-Software/modules/glossary-frontend/glossary-api\|glossaryApi]] | internal | Fetch term info for tooltips (via provider) |
| moduleEventBus | internal | Subscribe to entry-saved/deleted events for cache invalidation |

## Side Effects

- Creates and manages DOM elements (tooltip `div.glossary-tooltip`) attached to the editor container.
- Subscribes to DOM events (`mouseover`, `mouseout`, `click`) on the editor view's DOM element.
- Fetches data from the glossary API (cached — subsequent hovers for the same term use the in-memory cache until invalidated).
- Subscribes to moduleEventBus events for cache clearing.
