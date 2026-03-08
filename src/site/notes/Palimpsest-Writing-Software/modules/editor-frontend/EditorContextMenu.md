---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/editor-context-menu/","title":"EditorContextMenu.svelte","tags":["file","editor-frontend","svelte"],"updated":"2026-03-05T05:32:41.532-07:00"}
---


# `EditorContextMenu.svelte`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/EditorContextMenu.svelte`

> [!NOTE] Positioned context menu component for the ProseMirror editor. Renders context-aware menu items including formatting toggles, table/list operations, spelling suggestions, and project tools (Add to Glossary, Create Comment).

## Props

| Prop | Type | Required | Default | Description |
| ---- | ---- | -------- | ------- | ----------- |
| `x` | `number` | yes | — | Screen X coordinate |
| `y` | `number` | yes | — | Screen Y coordinate |
| `selectedText` | `string` | yes | — | Currently selected text |
| `context` | `EditorContext` | yes | — | Click location context flags |
| `activeStates` | `ActiveStates` | yes | — | Current mark active states (bold, italic, etc.) |
| `isMisspelled` | `boolean` | no | `false` | Whether the clicked word is misspelled |
| `misspelledWord` | `string` | no | `''` | The misspelled word |
| `spellingSuggestions` | `string[]` | no | `[]` | Spelling suggestions |
| `onaction` | callback | yes | — | Called with `(action, data?)` when a menu item is clicked |
| `onclose` | callback | yes | — | Called when the menu should close |

## `ActiveStates` Interface

| Field | Type | Description |
| ----- | ---- | ----------- |
| `bold` | `boolean` | Bold mark active |
| `italic` | `boolean` | Italic mark active |
| `underline` | `boolean` | Underline mark active |
| `strikethrough` | `boolean` | Strikethrough mark active |
| `code` | `boolean` | Inline code mark active |
| `highlight` | `boolean` | Highlight mark active |

## Menu Item Categories

Menu items are built dynamically using `$derived.by()`:

### Spelling Items (if `isMisspelled`)

Shown at the top of the menu:

- Up to 5 spelling suggestions (action: `suggest:<word>`)
- "No suggestions" disabled item (if no suggestions available)
- Divider
- "Add to Dictionary" (action: `addToDictionary`)
- Divider before formatting items

### Base Formatting Items (always shown)

| Action | Label | Icon |
| ------ | ----- | ---- |
| `bold` | Bold | Bold |
| `italic` | Italic | Italic |
| `underline` | Underline | Underline |
| `strikethrough` | Strikethrough | Strikethrough |
| `code` | Code | Code |
| `highlight` | Highlight | Palette |

Active marks show a colored accent dot indicator.

### Context-Specific Items

| Context | Items |
| ------- | ----- |
| `inTable` | Add Row Below, Add Column Right, Delete Row, Delete Column |
| `inList` | Increase Indent, Decrease Indent |
| `inBlockquote` | Remove Quote |
| `inDetails` | Toggle Details |

### Project Items (if text selected)

| Action | Label | Icon |
| ------ | ----- | ---- |
| `glossary` | Add to Glossary | BookPlus |
| `comment` | Create Comment | MessageSquarePlus |

> [!TIP] The "Add to Glossary" and "Create Comment" items are currently stubs — they emit events to the parent component but the full workflows are not yet implemented.

## Behavior

### Positioning

The menu uses `position: fixed` and adjusts coordinates to stay within the viewport:

- Right overflow: shifts left to keep 10px from edge
- Bottom overflow: shifts up to keep 10px from edge
- Ensures menu stays at least 10px from left and top edges

Position is recalculated via `requestAnimationFrame` after initial render.

### Keyboard Navigation

Document-level `keydown` listener (not element-focused to avoid ProseMirror focus theft):

| Key | Action |
| --- | ------ |
| `Escape` | Close menu |
| `ArrowDown` | Move to next item |
| `ArrowUp` | Move to previous item |
| `Enter` | Execute focused item |

> [!WARNING] The menu intentionally does **not** focus itself. Stealing focus from ProseMirror causes the browser to corrupt the DOM selection when the menu is destroyed, resulting in the entire paragraph getting selected.

### Click Outside

Document-level `mousedown` listener closes the menu when clicking outside the menu element.

### MouseDown Prevention

The menu container uses `onmousedown={(e) => e.preventDefault()}` to prevent ProseMirror from starting a selection when the user clicks a menu item.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `onMount`, `onDestroy` | `svelte` | Lifecycle hooks |
| `createLogger` | `$lib/services/loggerService` | Logger (`'context-menu'`) |
| 16 Lucide icons | `lucide-svelte` | Menu item icons |
| `EditorContext` | `./proseMirror/plugins/contextMenu` | Context type |

## Side Effects

- Adds document-level `mousedown` and `keydown` listeners on mount
- Removes listeners on destroy
- Logs action IDs on menu item click
