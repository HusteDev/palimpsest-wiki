---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/typography/font-manager/","title":"Font Manager","tags":["feature","fonts","typography","core"],"updated":"2026-03-07T17:43:45.461-07:00"}
---


# Font Manager

> [!NOTE] Modal interface for browsing, downloading, and managing fonts from Google Fonts. Integrates with project metadata to track font usage and auto-download missing fonts when opening projects.

## Overview

The Font Manager provides authors with access to the full Google Fonts library while maintaining offline-first capabilities. When fonts are downloaded, they are stored locally and embedded into PDF exports to ensure consistent typography regardless of the reader's system fonts.

Key capabilities:

- **Google Fonts integration:** Browse and search the entire Google Fonts library (1500+ font families)
- **Local font storage:** Downloaded fonts are stored in the Tauri data directory for offline use
- **Project font tracking:** Automatically tracks which fonts are used in each project's content
- **Auto-download on project open:** Missing project fonts are downloaded automatically when opening a project
- **Keyboard-driven navigation:** Full keyboard support for accessibility and power users

## Architecture

The font system follows a layered architecture:

```
Component Layer    FontManagerModal.svelte — UI for browsing and managing fonts
     |
Service Layer      fontService.ts — font download, installation, project font checks
     |
Tracking Layer     fontTrackingService.ts — monitors content changes, syncs to backend
     |
Backend Layer      /api/projects/{slug}/fonts — project font metadata storage
     |
Storage Layer      Tauri data directory — local font file storage
```

### Layer Responsibilities

| Layer | Responsibility |
| ----- | -------------- |
| **Component** | Renders modal UI with search, font preview, download/delete actions, keyboard navigation |
| **Service** | Handles font download from CDN, installation to local storage, project font availability checks |
| **Tracking** | Monitors ProseMirror document changes, detects font usage, batches updates to backend |
| **Backend** | Persists project font metadata (`fontSlug -> contentIds[]`) in SQLite via Ent ORM |
| **Storage** | Stores downloaded font files (`.ttf`) in `{data_dir}/fonts/{font-slug}/` |

## Font Manager Modal

The Font Manager Modal is accessed from the Header component via a toolbar button. It provides a unified interface for font discovery and management.

### User Interface

| Section | Content |
| ------- | ------- |
| **Search bar** | Text input for filtering fonts by name; auto-focused on modal open |
| **Font tabs** | Toggle between "Installed" fonts and "Available" fonts |
| **Font list** | Scrollable list of font families with preview text showing the font |
| **Font actions** | Download button (available) or Delete button (installed) per font |
| **Status indicators** | Download progress bar, installation status icons |

### Search and Discovery

1. User opens Font Manager modal (keyboard: Header toolbar button)
2. Search input receives auto-focus
3. User types font name (e.g., "Roboto", "Merriweather")
4. Font list filters in real-time using case-insensitive matching
5. Results show font preview using the actual font (lazy-loaded from Google Fonts CSS)

### Downloading Fonts

When a user clicks the download button on an available font:

1. Font variants are fetched from Google Fonts API (regular, bold, italic, bold-italic)
2. Each variant's TTF file is downloaded to `{data_dir}/fonts/{font-slug}/`
3. Progress indicator shows download status
4. Upon completion, font appears in "Installed" tab

### Deleting Fonts

Installed fonts can be deleted with confirmation:

1. User selects a font in the Installed tab
2. User clicks Delete button or presses `Delete`/`Backspace` key
3. Confirmation dialog appears: "Delete {font-name}?"
4. On confirm, font files are removed from local storage
5. Font disappears from Installed tab

> [!WARNING] Deleting a font that is used in project content will cause that content to fall back to the default font until the font is re-downloaded.

## Keyboard Navigation

The Font Manager supports full keyboard navigation for accessibility and power user workflows:

| Key | Action |
| --- | ------ |
| `ArrowUp` | Move selection to previous font in list |
| `ArrowDown` | Move selection to next font in list |
| `Enter` | Download selected font (if available) |
| `Delete` / `Backspace` | Open delete confirmation for selected font (if installed) |
| `Escape` | Cancel delete confirmation / close modal |
| `Tab` | Navigate between search input and font list |

### Focus Management

- Search input receives auto-focus when modal opens
- Arrow key navigation scrolls the selected font into view
- Selected font has visual highlight with focus ring
- Focus is trapped within modal while open

### ARIA Accessibility

| Element | ARIA Role/Attribute |
| ------- | ------------------- |
| Font list container | `role="listbox"` |
| Font item | `role="option"`, `aria-selected` |
| Search input | Standard text input with label |
| Delete confirmation | `role="alertdialog"` |

## Project Font Tracking

The font tracking service automatically monitors content changes and maintains a mapping of which fonts are used in each project.

### How Font Tracking Works

1. **Content observation:** `fontTrackingService` subscribes to ProseMirror transactions
2. **Font detection:** On each change, the service scans for `font-family` styles in the document
3. **Batched updates:** Font usage changes are batched (500ms debounce) to minimize API calls
4. **Backend sync:** Pending updates are sent to `PATCH /api/projects/{slug}/fonts`
5. **Project metadata:** Backend stores `projectFonts: { fontSlug: [contentId, ...] }` in Project entity

### Font Tracking Data Model

```typescript
// Project fonts map stored in backend
interface ProjectFonts {
  [fontSlug: string]: string[]; // Array of content IDs using this font
}

// Example
{
  "roboto": ["chapter-1", "chapter-2"],
  "merriweather": ["chapter-3"]
}
```

### Auto-Download on Project Open

When a project is opened, the system ensures all required fonts are available:

1. **Project load:** `loadProject()` fetches project data including `projectFonts`
2. **Font check:** `ensureProjectFonts()` compares project fonts against installed fonts
3. **Missing detection:** Fonts not in local storage are identified
4. **Auto-download:** Missing fonts are downloaded automatically in background
5. **Progress feedback:** Download progress shown in UI (if significant download required)
6. **Graceful fallback:** If download fails, content renders with fallback font and warning logged

```typescript
// Called in editor/[slug]/+page.svelte loadProject()
if (project.projectFonts && Object.keys(project.projectFonts).length > 0) {
  ensureProjectFonts(project.projectFonts).then((result) => {
    if (!result.allAvailable) {
      log.warn('[loadProject] Some project fonts could not be downloaded',
        { missing: result.missing });
    }
  });
}
```

## Default Fonts

The application includes a set of default fonts that are always available:

| Font | Category | Purpose |
| ---- | -------- | ------- |
| Lora | Serif | Default body text font |
| Inter | Sans-serif | UI font and clean body option |
| Fira Code | Monospace | Code blocks and inline code |
| Spectral | Serif | Literary body text alternative |
| Merriweather | Serif | Highly readable body text |
| Open Sans | Sans-serif | Clean, modern body text |

> [!TIP] Default fonts are bundled with the application and do not require download. They are identified by the `isDefaultFont()` utility function.

## Backend API

### Get Project Fonts

```
GET /api/projects/{slug}/fonts
```

**Response:**
```json
{
  "fonts": {
    "roboto": ["content-id-1", "content-id-2"],
    "merriweather": ["content-id-3"]
  }
}
```

### Update Project Fonts

```
PATCH /api/projects/{slug}/fonts
```

**Request Body:**
```json
{
  "fonts": {
    "roboto": ["content-id-1", "content-id-2"]
  }
}
```

**Behavior:** Merges provided fonts with existing project fonts. To remove a font entirely, set its value to an empty array.

## File Storage

Downloaded fonts are stored in the Tauri application data directory:

```
{APP_DATA}/fonts/
├── roboto/
│   ├── roboto-regular.ttf
│   ├── roboto-bold.ttf
│   ├── roboto-italic.ttf
│   └── roboto-bolditalic.ttf
├── merriweather/
│   ├── merriweather-regular.ttf
│   └── ...
└── open-sans/
    └── ...
```

Font files are TTF format for maximum compatibility with PDF generation libraries.

## Integration Points

### Editor Integration

- [[Palimpsest-Writing-Software/features/editor/overview\|ProseMirror Editor]] applies font-family styles to text
- `fontTrackingService` monitors editor transactions for font usage changes
- Font preview in editor respects installed fonts

### PDF Export Integration

- PDF generation reads installed font files from local storage
- Fonts are embedded into PDF for portable rendering
- Missing fonts fall back to default with warning

### Print Modal Integration

- [[modules/print-frontend/PrintModal\|Print Modal]] shows available fonts for print styling
- Font selection limited to installed fonts
- Print preview renders with selected font

## Security

- Font files are downloaded only from trusted Google Fonts CDN endpoints
- Downloaded files are validated by file extension and MIME type
- No arbitrary font uploads from users (Google Fonts whitelist only)
- Local storage path is sandboxed within Tauri data directory

## Performance

- **Lazy font loading:** Font previews in modal load CSS on-demand, not all at once
- **Batched tracking updates:** Font usage changes are batched to reduce API calls (500ms debounce)
- **Background download:** Project font auto-download runs asynchronously, non-blocking
- **Indexed font list:** Font list uses virtualization for large font counts (1500+ fonts)

## Logging

| Logger Name | File | Purpose |
| ----------- | ---- | ------- |
| `fonts` | `fontService.ts` | Font download, installation, project font checks |
| `font-tracking` | `fontTrackingService.ts` | Font usage detection, sync status |
| `font-manager` | `FontManagerModal.svelte` | User actions, keyboard navigation |

## Related

- [[Palimpsest-Writing-Software/_index/moc-core-features\|MOC: Core Features]]
- [[Palimpsest-Writing-Software/features/editor/overview\|Feature: ProseMirror Editor]]
- [[modules/fonts-frontend/fontService\|Module: fontService]]
- [[modules/fonts-frontend/fontTrackingService\|Module: fontTrackingService]]
