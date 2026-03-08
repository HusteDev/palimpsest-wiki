---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/editor-frontend/file-handler/","title":"fileHandler.ts","tags":["file","editor-frontend","prosemirror","plugin"],"updated":"2026-03-05T05:32:41.632-07:00"}
---


# `fileHandler.ts`

**Module:** [[Palimpsest-Writing-Software/modules/editor-frontend/overview\|Editor Frontend]]
**Path:** `src/lib/components/editor/proseMirror/plugins/fileHandler.ts`

> [!NOTE] ProseMirror plugin for handling drag-and-drop and clipboard paste of image files. Converts images to base64 data URLs and inserts them as inline image nodes.

## Exports

| Name | Kind | Description |
| ---- | ---- | ----------- |
| `FileHandlerOptions` | interface | Configuration for the file handler plugin |
| `fileHandler` | function | Factory that creates the file handler plugin |

## `FileHandlerOptions` Interface

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| `allowedMimeTypes` | `string[]` | `['image/png', 'image/jpeg', 'image/gif', 'image/webp', 'image/svg+xml']` | MIME types accepted for insertion |
| `onDrop` | `(files, pos, view) => boolean` | `defaultImageHandler` | Custom handler for dropped files |
| `onPaste` | `(files, view) => boolean` | `defaultImageHandler(files, null, view)` | Custom handler for pasted files |

## Behavior

### Drag-and-Drop

1. **`dragover`:** Prevents default browser behavior (enables drop)
2. **`drop`:** Extracts files from `event.dataTransfer.files`, filters by `allowedMimeTypes`, converts mouse coordinates to document position via `view.posAtCoords()`, calls `onDrop` handler

### Clipboard Paste

1. **`paste`:** Extracts files from `event.clipboardData.files`, filters by `allowedMimeTypes`, calls `onPaste` handler

### Default Image Handler

The built-in `defaultImageHandler` function:

1. Iterates files that match `image/*` MIME type
2. Reads each file as base64 data URL via `FileReader.readAsDataURL()`
3. Creates an `image` node via `schema.nodes.image.create({ src: base64, alt: file.name, title: file.name })`
4. Inserts at the drop position (for drops) or replaces current selection (for pastes)
5. Logs errors via the `'file-handler'` logger

## Decision Points

| Condition | Branch |
| --------- | ------ |
| No files in transfer/clipboard | Return `false` (don't handle) |
| No files pass MIME filter | Return `false` |
| Drop position not found | Return `false` |
| File is an image | Convert to base64 and insert |
| File is not an image | Skip silently |
| FileReader error | Log error, skip file |

> [!WARNING] Images are inserted as **base64 data URLs**, which embeds the full image data in the document JSON. This can significantly increase document size for large images. The Pro license will introduce cloud storage for image assets.

## Imports / Dependencies

| Import | Source | Purpose |
| ------ | ------ | ------- |
| `Plugin` | `prosemirror-state` | Plugin factory |
| `EditorView` | `prosemirror-view` | View for dispatch and coordinate conversion |
| `schema` | `../schema` | Image node type reference |
| `createLogger` | `$lib/services/loggerService` | Logger (`'file-handler'`) |

## Side Effects

- Calls `event.preventDefault()` when handling a valid drop/paste
- Dispatches transactions to insert image nodes into the document
