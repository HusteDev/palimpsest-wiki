---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/modules/settings-frontend/overview/","title":"Settings Frontend Module","tags":["module","settings-frontend"],"updated":"2026-03-05T08:56:12.410-07:00"}
---


# Settings Frontend

> [!NOTE] Svelte store-based settings layer providing a read-only `systemSettings` store for infrastructure configuration and a mutable `userPreferences` store for personal settings, with debounced panel layout saves, floating panel lifecycle management, panel collapse/expand (mini mode), field migration from legacy formats, and derived stores for convenient reactive access.

## Responsibilities

- Load system settings and user preferences from the backend API (`GET /api/settings`)
- Provide read-only reactive access to system configuration via `systemSettings` store
- Provide mutable reactive access to user preferences via `userPreferences` store
- Save preference changes to backend via `PUT /api/settings` with automatic serialization
- Debounce rapid panel layout and UI state changes to prevent excessive API calls
- Manage floating panel lifecycle: pop-out, dock, move, resize, minimize, close, bring-to-front
- Manage panel collapse/expand (mini mode) for docked panels
- Migrate legacy field formats on load (addons → components, missing collapsedPanels, missing floating)
- Normalize floating panel z-indices to prevent overflow approaching modal z-index
- Provide convenience getters and derived stores for common settings checks

## Public API Surface

### systemSettings Store

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `systemSettings.subscribe` | store | Reactive subscription to `SystemSettings` |
| `systemSettings.isLoaded` | store | Whether initial load is complete |
| `systemSettings.load()` | method | Fetch system settings from backend API |
| `systemSettings.devTools` | getter | Synchronous snapshot of devTools flag |
| `systemSettings.logging` | getter | Synchronous snapshot of `LoggingConfig` |
| `devToolsEnabled` | derived | Reactive derived store for devTools check |

### userPreferences Store

| Export | Kind | Description |
| ------ | ---- | ----------- |
| `userPreferences.subscribe` | store | Reactive subscription to `UserPreferences` |
| `userPreferences.isLoading` | store | Whether a load/save operation is in progress |
| `userPreferences.error` | store | Last error message (or null) |
| `userPreferences.load()` | method | Fetch preferences from backend API |
| `userPreferences.updateTheme(theme)` | method | Update and save theme |
| `userPreferences.updateFont(font)` | method | Update and save font |
| `userPreferences.updatePanelLayout(updates)` | method | Update panel dimensions (debounced save) |
| `userPreferences.togglePanel(panel)` | method | Toggle panel visibility (immediate save) |
| `userPreferences.updateEditor(settings)` | method | Update editor settings (immediate save) |
| `userPreferences.updateWordCount(settings)` | method | Update word count settings (immediate save) |
| `userPreferences.setLastProject(slug)` | method | Record last opened project |
| `userPreferences.setDisableAutoNumbering(disabled)` | method | Toggle auto-numbering |
| `userPreferences.updateUIState(updates)` | method | Update hidden UI state (debounced save) |
| `userPreferences.reset()` | method | Reset all preferences to defaults |
| **Floating Panel Actions** | | |
| `userPreferences.popOutPanel(componentId, source, bounds?)` | method | Pop docked component into floating panel |
| `userPreferences.dockPanel(panelId, target)` | method | Dock floating panel back to position |
| `userPreferences.updateFloatingPanel(panelId, updates)` | method | Update position/size (debounced save) |
| `userPreferences.closeFloatingPanel(panelId)` | method | Remove floating panel |
| `userPreferences.bringToFront(panelId)` | method | Raise z-index to top |
| `userPreferences.toggleMinimize(panelId)` | method | Toggle minimize state |
| `userPreferences.getFloatingPanel(panelId)` | method | Get floating panel by ID |
| **Panel Collapse Actions** | | |
| `userPreferences.collapsePanel(position)` | method | Collapse panel to mini mode |
| `userPreferences.expandPanel(position, activeTabId?)` | method | Expand from mini mode |
| `userPreferences.togglePanelCollapsed(position)` | method | Toggle collapsed state |
| `userPreferences.isPanelCollapsed(position)` | method | Check if panel is collapsed |

### Types

| Type | Source | Description |
| ---- | ------ | ----------- |
| `SystemSettings` | systemSettings.ts | System configuration interface |
| `LoggingConfig` | systemSettings.ts | Logging configuration interface |
| `ComponentLogging` | systemSettings.ts | Per-component logging override |
| `UserPreferences` | preferences.ts | Complete user preferences interface |
| `EditorSettings` | preferences.ts | Editor-specific settings |
| `WordCountSettings` | preferences.ts | Word count display settings |
| `UIState` | preferences.ts | Dynamic UI state map |
| `PanelConfig` | preferences.ts | Docked panel configuration |
| `PanelLayout` | preferences.ts | Full panel layout (docked + floating) |
| `FloatingPanelInstance` | preferences.ts | Floating panel position and state |

## Internal Structure

**System Settings Store** — Read-only infrastructure configuration:
- `[[modules/settings-frontend/systemSettings|systemSettings.ts]]` — `SystemSettings` interface, `LoggingConfig` interface, defaults, `loadSettings()`, derived `devToolsEnabled` store

**User Preferences Store** — Mutable personal settings with full panel management:
- `[[modules/settings-frontend/preferences|preferences.ts]]` — `UserPreferences` interface, all nested types, default values, `migratePanelLayout()`, CRUD actions, debounced saves, floating panel lifecycle, panel collapse/expand, z-index normalization

## Dependencies

| Dependency | Internal / External | Purpose |
| ---------- | ------------------- | ------- |
| `svelte/store` | external | `writable`, `derived`, `get` for reactive stores |
| `$lib/api/client` | internal | `apiClient` for HTTP requests |
| `$lib/config` | internal | `PANEL_DEBOUNCE` timing constant |
| `$lib/services/loggerService` | internal | `createLogger` for structured logging |

## Integration Points

- **Startup** — Both `systemSettings.load()` and `userPreferences.load()` are called during app initialization
- **Theme engine** — `themeEngine` subscribes to `userPreferences` for theme changes
- **Panel system** — Panel components read `panelLayout` and call `updatePanelLayout()`, `popOutPanel()`, `dockPanel()`, etc.
- **Editor** — Editor page reads `editor` settings for spellcheck, autosave, word wrap
- **Content tree** — Tree component reads `uiState.expandedTreeIds` for expansion state
- **Status bar** — Status bar reads `wordCount` settings for display configuration
- **Logging** — `loggerService.ts` reads `systemSettings.logging` to configure frontend log levels

## Logging

- `systemSettings.ts`: `createLogger('settings')`
- `preferences.ts`: `createLogger('preferences')`

## Related

- [[Palimpsest-Writing-Software/architecture/settings/overview\|Settings System]]
- [[Palimpsest-Writing-Software/modules/settings-backend/overview\|Settings Backend Module]]
- [[Palimpsest-Writing-Software/api/settings/overview\|Settings API Service]]
- [[Palimpsest-Writing-Software/features/panel-system/overview\|Panel System]]
