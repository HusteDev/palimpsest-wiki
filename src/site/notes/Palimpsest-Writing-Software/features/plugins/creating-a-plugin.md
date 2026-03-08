---
{"dg-publish":true,"permalink":"/palimpsest-writing-software/features/plugins/creating-a-plugin/","title":"Creating a Plugin: Step-by-Step Guide","tags":["feature","plugins","tutorial","guide"],"updated":"2026-03-05T10:11:08.463-07:00"}
---


# Creating a Palimpsest Plugin: Step-by-Step Guide

> [!NOTE] This guide walks through creating a complete Palimpsest plugin from scratch, using an Excalidraw whiteboard integration as the working example. By the end, you'll have a fully functional plugin that adds a new "Drawing" content type to Palimpsest.

## Table of Contents

1. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Prerequisites\|#Prerequisites]]
2. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Understanding the Plugin Architecture\|#Understanding the Plugin Architecture]]
3. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 1 — Project Setup\|#Phase 1 — Project Setup]]
4. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 2 — Create the Plugin Manifest\|#Phase 2 — Create the Plugin Manifest]]
5. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 3 — Build the Frontend Bundle\|#Phase 3 — Build the Frontend Bundle]]
6. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 4 — Create the Renderer Component\|#Phase 4 — Create the Renderer Component]]
7. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 5 — Implement Lifecycle Hooks\|#Phase 5 — Implement Lifecycle Hooks]]
8. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 6 — Build and Bundle\|#Phase 6 — Build and Bundle]]
9. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 7 — Install the Plugin\|#Phase 7 — Install the Plugin]]
10. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 8 — Test and Debug\|#Phase 8 — Test and Debug]]
11. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Complete Source Code\|#Complete Source Code]]
12. [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Troubleshooting\|#Troubleshooting]]

---

## Prerequisites

Before starting, ensure you have:

| Requirement | Version | Purpose |
| ----------- | ------- | ------- |
| Node.js | 18+ | JavaScript runtime |
| pnpm | 8+ | Package manager (preferred) |
| Palimpsest | 1.0.0+ | The application you're extending |
| Text Editor | Any | VS Code recommended |

> [!TIP] If you don't have pnpm installed, run `npm install -g pnpm`.

You should also have a basic understanding of:
- TypeScript
- Svelte 5 (runes: `$state`, `$derived`, `$effect`)
- npm/pnpm package management

---

## Understanding the Plugin Architecture

Before diving in, understand what we're building:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Palimpsest Application                       │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐     ┌─────────────────────────────────┐   │
│  │  Plugin System  │────▶│  ContentRendererRegistry        │   │
│  └─────────────────┘     │  (maps content type → renderer) │   │
│           │              └─────────────────────────────────┘   │
│           │                              │                      │
│           ▼                              ▼                      │
│  ┌─────────────────┐     ┌─────────────────────────────────┐   │
│  │ Plugin Manifest │     │  Your Renderer Component        │   │
│  │ (JSON config)   │     │  (Svelte component)             │   │
│  └─────────────────┘     └─────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Key concepts:**
- **Manifest** — A JSON file (`palimpsest-plugin.json`) declaring your plugin's capabilities
- **Renderer** — A Svelte component that displays your custom content type
- **Lifecycle hooks** — Functions called at specific points (content create, load, save)
- **Plugin API** — Sandboxed APIs provided to your plugin for storage, notifications, etc.

---

## Phase 1 — Project Setup

### Step 1.1: Create the Plugin Directory

Plugins live in Palimpsest's data directory. Create a new folder:

**Windows:**
```bash
mkdir "%USERPROFILE%\Documents\Palimpsest\Plugins\excalidraw"
cd "%USERPROFILE%\Documents\Palimpsest\Plugins\excalidraw"
```

**macOS/Linux:**
```bash
mkdir -p ~/Documents/Palimpsest/Plugins/excalidraw
cd ~/Documents/Palimpsest/Plugins/excalidraw
```

### Step 1.2: Initialize the Project

```bash
pnpm init
```

Edit the generated `package.json`:

```json
{
  "name": "palimpsest-plugin-excalidraw",
  "version": "1.0.0",
  "type": "module",
  "description": "Excalidraw whiteboard integration for Palimpsest",
  "main": "dist/bundle.js",
  "scripts": {
    "dev": "vite build --watch",
    "build": "vite build",
    "check": "svelte-check --tsconfig ./tsconfig.json"
  },
  "keywords": ["palimpsest", "plugin", "excalidraw"],
  "author": "Your Name",
  "license": "MIT"
}
```

### Step 1.3: Install Dependencies

```bash
# Core dependencies
pnpm add @excalidraw/excalidraw react react-dom

# Development dependencies
pnpm add -D vite svelte @sveltejs/vite-plugin-svelte typescript svelte-check
pnpm add -D @types/react @types/react-dom @tsconfig/svelte
```

> [!IMPORTANT] Excalidraw is a React library. We'll need to create a wrapper to use it in our Svelte component. This is covered in [[Palimpsest-Writing-Software/features/plugins/creating-a-plugin#Phase 4 — Create the Renderer Component\|#Phase 4 — Create the Renderer Component]].

### Step 1.4: Create the File Structure

```bash
mkdir -p src dist
```

Your project should now look like:

```
excalidraw/
├── package.json
├── node_modules/
├── src/              ← Source code goes here
│   ├── index.ts
│   ├── ExcalidrawRenderer.svelte
│   └── ExcalidrawWrapper.tsx
├── dist/             ← Built output goes here
│   └── bundle.js     ← (generated by build)
└── palimpsest-plugin.json
```

---

## Phase 2 — Create the Plugin Manifest

Create `palimpsest-plugin.json` in your plugin root:

```json
{
  "name": "excalidraw",
  "displayName": "Excalidraw",
  "version": "1.0.0",
  "palimpsestVersion": "^1.0.0",
  "description": "Add Excalidraw whiteboard drawings to your writing projects. Create diagrams, mind maps, and visual notes alongside your content.",
  "author": {
    "name": "Your Name",
    "email": "you@example.com",
    "url": "https://github.com/yourusername"
  },
  "license": "MIT",
  "repository": "https://github.com/yourusername/palimpsest-plugin-excalidraw",
  "tier": "core",
  "capabilities": {
    "contentTypes": true,
    "panels": false,
    "contextMenuItems": false,
    "jobHandlers": false,
    "restEndpoints": false
  },
  "permissions": {
    "fileSystem": false,
    "network": false,
    "localStorage": true,
    "clipboard": true
  },
  "entryPoints": {
    "frontend": "dist/bundle.js"
  },
  "contentTypes": [
    {
      "id": "drawing",
      "name": "Excalidraw Drawing",
      "icon": "PenTool",
      "color": "#6965db",
      "hasContent": true,
      "canHaveChildren": false,
      "canBeChildOf": "*",
      "renderer": "ExcalidrawRenderer",
      "dataSchema": {
        "type": "object",
        "properties": {
          "elements": { "type": "array" },
          "appState": { "type": "object" },
          "files": { "type": "object" }
        },
        "required": ["elements"]
      }
    }
  ]
}
```

### Manifest Field Reference

| Field | Required | Description |
| ----- | -------- | ----------- |
| `name` | Yes | Unique identifier. Lowercase, starts with letter, `[a-z0-9-]` only. |
| `displayName` | Yes | Human-readable name shown in UI. |
| `version` | Yes | Semantic version (e.g., `1.0.0`). |
| `palimpsestVersion` | Yes | Compatible Palimpsest version range. |
| `description` | Yes | Brief description (shown in plugin settings). |
| `author` | No | Author info (name, email, url). |
| `tier` | Yes | `"core"` (free) or `"pro"` (paid features). |
| `capabilities` | Yes | Which extension points the plugin uses. |
| `permissions` | Yes | What system access the plugin needs. |
| `entryPoints.frontend` | Yes | Path to the JavaScript bundle. |
| `contentTypes` | Conditional | Required if `capabilities.contentTypes` is `true`. |

### Content Type Definition

| Field | Description |
| ----- | ----------- |
| `id` | Unique ID within this plugin (e.g., `"drawing"`). Combined with plugin name to form full ID: `plugin:excalidraw:drawing`. |
| `name` | Display name in the "Create Content" modal. |
| `icon` | Lucide icon name (see [lucide.dev/icons](https://lucide.dev/icons)). |
| `color` | Optional hex color for visual distinction. |
| `hasContent` | Must be `true` — plugin content types always have content. |
| `canHaveChildren` | Must be `false` — plugin content cannot have children. |
| `canBeChildOf` | `"*"` for any parent, or array of specific content type IDs. |
| `renderer` | Name of the exported Svelte component. |
| `dataSchema` | Optional JSON Schema for content validation. |

---

## Phase 3 — Build the Frontend Bundle

### Step 3.1: Create TypeScript Configuration

Create `tsconfig.json`:

```json
{
  "extends": "@tsconfig/svelte/tsconfig.json",
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "types": ["svelte"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Step 3.2: Create Vite Configuration

Create `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import { svelte } from '@sveltejs/vite-plugin-svelte';

export default defineConfig({
  plugins: [
    svelte({
      compilerOptions: {
        // Enable runes (Svelte 5)
        runes: true
      }
    })
  ],
  build: {
    lib: {
      entry: 'src/index.ts',
      formats: ['es'],
      fileName: () => 'bundle.js'
    },
    outDir: 'dist',
    emptyDirOnBuild: true,
    rollupOptions: {
      // Don't bundle Svelte internals — Palimpsest provides them
      external: ['svelte', 'svelte/internal', 'svelte/store'],
      output: {
        // Ensure a single bundle file
        inlineDynamicImports: true
      }
    }
  },
  // Optimize Excalidraw's dependencies
  optimizeDeps: {
    include: ['@excalidraw/excalidraw', 'react', 'react-dom']
  }
});
```

### Step 3.3: Create Svelte Configuration

Create `svelte.config.js`:

```javascript
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

export default {
  preprocess: vitePreprocess(),
  compilerOptions: {
    runes: true
  }
};
```

---

## Phase 4 — Create the Renderer Component

The renderer component is the heart of your plugin. It receives content data and renders it.

### Step 4.1: Understand the Renderer Props

Your renderer receives these props from Palimpsest:

```typescript
interface PluginRendererProps {
  /** The content item being rendered */
  content: Content;
  /** The content data (from Content.content field) */
  data: unknown;
  /** Whether editing is allowed (false if content is locked) */
  editable: boolean;
  /** Callback when content data changes */
  onupdate: (data: unknown) => void;
}
```

### Step 4.2: Create the React Wrapper for Excalidraw

Since Excalidraw is a React component, we need a wrapper. Create `src/ExcalidrawWrapper.tsx`:

```tsx
// src/ExcalidrawWrapper.tsx
//
// React wrapper for Excalidraw component.
// This bridges React (Excalidraw) and Svelte (Palimpsest).

import React, { useEffect, useRef, useCallback } from 'react';
import { Excalidraw } from '@excalidraw/excalidraw';
import type {
  ExcalidrawElement,
  AppState,
  BinaryFiles
} from '@excalidraw/excalidraw/types/types';
import type { ExcalidrawImperativeAPI } from '@excalidraw/excalidraw/types/types';

/**
 * Props for the ExcalidrawWrapper component.
 */
export interface ExcalidrawWrapperProps {
  /** Initial drawing elements */
  initialElements: readonly ExcalidrawElement[];
  /** Initial app state (view settings) */
  initialAppState: Partial<AppState>;
  /** Initial binary files (images, etc.) */
  initialFiles: BinaryFiles;
  /** Whether the drawing is editable */
  editable: boolean;
  /** Callback when the drawing changes */
  onChange: (
    elements: readonly ExcalidrawElement[],
    appState: AppState,
    files: BinaryFiles
  ) => void;
}

/**
 * React component that wraps Excalidraw.
 * Handles initialization, change events, and read-only mode.
 */
export function ExcalidrawWrapper({
  initialElements,
  initialAppState,
  initialFiles,
  editable,
  onChange
}: ExcalidrawWrapperProps) {
  const excalidrawRef = useRef<ExcalidrawImperativeAPI | null>(null);

  // Debounce changes to avoid excessive updates
  const changeTimeoutRef = useRef<NodeJS.Timeout | null>(null);

  const handleChange = useCallback(
    (elements: readonly ExcalidrawElement[], appState: AppState, files: BinaryFiles) => {
      // Skip if not editable
      if (!editable) return;

      // Debounce: wait 500ms after last change before notifying
      if (changeTimeoutRef.current) {
        clearTimeout(changeTimeoutRef.current);
      }

      changeTimeoutRef.current = setTimeout(() => {
        // Filter out deleted elements for cleaner storage
        const activeElements = elements.filter((el) => !el.isDeleted);

        // Strip volatile app state (selection, cursor, etc.)
        const persistentAppState: Partial<AppState> = {
          viewBackgroundColor: appState.viewBackgroundColor,
          gridSize: appState.gridSize,
          zenModeEnabled: appState.zenModeEnabled,
          theme: appState.theme
        };

        onChange(activeElements, persistentAppState as AppState, files);
      }, 500);
    },
    [editable, onChange]
  );

  // Cleanup timeout on unmount
  useEffect(() => {
    return () => {
      if (changeTimeoutRef.current) {
        clearTimeout(changeTimeoutRef.current);
      }
    };
  }, []);

  return (
    <div style={{ width: '100%', height: '100%' }}>
      <Excalidraw
        ref={(api) => {
          excalidrawRef.current = api;
        }}
        initialData={{
          elements: initialElements,
          appState: {
            ...initialAppState,
            // Ensure collaborators array exists
            collaborators: new Map()
          },
          files: initialFiles
        }}
        onChange={handleChange}
        viewModeEnabled={!editable}
        zenModeEnabled={false}
        gridModeEnabled={false}
        theme="light"
        langCode="en"
        UIOptions={{
          canvasActions: {
            changeViewBackgroundColor: editable,
            clearCanvas: editable,
            export: { saveFileToDisk: true },
            loadScene: editable,
            saveToActiveFile: false,
            toggleTheme: true
          }
        }}
      />
    </div>
  );
}

export default ExcalidrawWrapper;
```

### Step 4.3: Create the Svelte Renderer Component

Create `src/ExcalidrawRenderer.svelte`:

```svelte
<!-- src/ExcalidrawRenderer.svelte -->
<!--
  Svelte wrapper for the Excalidraw React component.
  This component is registered with Palimpsest's ContentRendererRegistry
  and receives content data via props.
-->
<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import React from 'react';
  import { createRoot, type Root } from 'react-dom/client';
  import { ExcalidrawWrapper, type ExcalidrawWrapperProps } from './ExcalidrawWrapper';
  import type { ExcalidrawElement, AppState, BinaryFiles } from '@excalidraw/excalidraw/types/types';

  /**
   * Content item from Palimpsest.
   */
  interface Content {
    id: string;
    title: string;
    slug: string;
    contentTypeId: string;
    isLocked: boolean;
    content?: unknown;
  }

  /**
   * Props received from Palimpsest's content rendering system.
   */
  interface Props {
    content: Content;
    data: ExcalidrawData | null;
    editable: boolean;
    onupdate: (data: ExcalidrawData) => void;
  }

  /**
   * Shape of Excalidraw data stored in the content.
   */
  interface ExcalidrawData {
    elements: readonly ExcalidrawElement[];
    appState: Partial<AppState>;
    files: BinaryFiles;
  }

  let { content, data, editable, onupdate }: Props = $props();

  // DOM reference for React root
  let containerEl: HTMLDivElement;
  let reactRoot: Root | null = null;

  // Default empty drawing
  const defaultData: ExcalidrawData = {
    elements: [],
    appState: {
      viewBackgroundColor: '#ffffff',
      gridSize: null
    },
    files: {}
  };

  // Get initial data, using defaults for missing fields
  let currentData = $derived<ExcalidrawData>({
    elements: data?.elements ?? defaultData.elements,
    appState: data?.appState ?? defaultData.appState,
    files: data?.files ?? defaultData.files
  });

  /**
   * Handle changes from Excalidraw.
   * Called with debounced updates from the React component.
   */
  function handleChange(
    elements: readonly ExcalidrawElement[],
    appState: AppState,
    files: BinaryFiles
  ): void {
    const newData: ExcalidrawData = {
      elements,
      appState: {
        viewBackgroundColor: appState.viewBackgroundColor,
        gridSize: appState.gridSize,
        zenModeEnabled: appState.zenModeEnabled,
        theme: appState.theme
      },
      files
    };
    onupdate(newData);
  }

  /**
   * Render the React component into the container.
   */
  function renderReact(): void {
    if (!containerEl) return;

    const props: ExcalidrawWrapperProps = {
      initialElements: currentData.elements,
      initialAppState: currentData.appState,
      initialFiles: currentData.files,
      editable,
      onChange: handleChange
    };

    const element = React.createElement(ExcalidrawWrapper, props);

    if (!reactRoot) {
      reactRoot = createRoot(containerEl);
    }
    reactRoot.render(element);
  }

  onMount(() => {
    renderReact();
  });

  onDestroy(() => {
    if (reactRoot) {
      reactRoot.unmount();
      reactRoot = null;
    }
  });

  // Re-render when editable state changes
  $effect(() => {
    if (containerEl && reactRoot) {
      // Track editable changes
      const _ = editable;
      renderReact();
    }
  });
</script>

<div class="excalidraw-renderer">
  {#if !editable}
    <div class="readonly-banner">
      <span>This drawing is locked and cannot be edited</span>
    </div>
  {/if}
  <div class="excalidraw-container" bind:this={containerEl}></div>
</div>

<style>
  .excalidraw-renderer {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    min-height: 500px;
    position: relative;
  }

  .readonly-banner {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    padding: 8px 16px;
    background: rgba(0, 0, 0, 0.05);
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    text-align: center;
    font-size: 12px;
    color: #666;
    z-index: 10;
  }

  .excalidraw-container {
    flex: 1;
    width: 100%;
    height: 100%;
  }

  /* Excalidraw's styles override - ensure it fills the container */
  .excalidraw-container :global(.excalidraw) {
    width: 100%;
    height: 100%;
  }

  .excalidraw-container :global(.excalidraw .excalidraw-container) {
    width: 100%;
    height: 100%;
  }
</style>
```

---

## Phase 5 — Implement Lifecycle Hooks

Lifecycle hooks let you customize content creation, loading, and saving.

### Step 5.1: Create the Plugin Entry Point

Create `src/index.ts`:

```typescript
// src/index.ts
//
// Plugin entry point for Palimpsest.
// Exports the renderer component and lifecycle hooks.

// Re-export the renderer component
// This name must match the "renderer" field in palimpsest-plugin.json
export { default as ExcalidrawRenderer } from './ExcalidrawRenderer.svelte';

// Types for lifecycle hooks
import type { ExcalidrawElement, AppState, BinaryFiles } from '@excalidraw/excalidraw/types/types';

/**
 * Excalidraw data structure stored in content.
 */
interface ExcalidrawData {
  elements: readonly ExcalidrawElement[];
  appState: Partial<AppState>;
  files: BinaryFiles;
}

/**
 * Plugin context provided by Palimpsest.
 */
interface PluginContext {
  pluginId: string;
  dataDir: string;
  settings: Record<string, unknown>;
  api: {
    content: {
      get: (projectSlug: string, slugPath: string[]) => Promise<unknown>;
      update: (projectSlug: string, slugPath: string[], data: unknown) => Promise<unknown>;
    };
    notifications: {
      success: (message: string) => void;
      error: (message: string) => void;
      info: (message: string) => void;
      warning: (message: string) => void;
    };
    storage: {
      get: <T>(key: string) => T | null;
      set: <T>(key: string, value: T) => void;
      remove: (key: string) => void;
    };
  };
}

/**
 * Content item from Palimpsest.
 */
interface Content {
  id: string;
  title: string;
  slug: string;
  contentTypeId: string;
  isLocked: boolean;
  content?: unknown;
}

/**
 * Input for creating new plugin content.
 */
interface CreatePluginContentInput {
  title: string;
  contentTypeId: string;
  content: unknown;
  customFields?: Record<string, unknown>;
}

/**
 * Lifecycle hooks for the Excalidraw plugin.
 */
export const lifecycle = {
  /**
   * Called when the plugin is enabled.
   * Use for initialization tasks.
   */
  async onEnable(ctx: PluginContext): Promise<void> {
    console.log('[excalidraw] plugin enabled');

    // You could restore user preferences here
    const savedTheme = ctx.api.storage.get<string>('preferred-theme');
    if (savedTheme) {
      console.log('[excalidraw] restored theme preference:', savedTheme);
    }
  },

  /**
   * Called when the plugin is disabled.
   * Use for cleanup tasks.
   */
  async onDisable(ctx: PluginContext): Promise<void> {
    console.log('[excalidraw] plugin disabled');
  },

  /**
   * Called when creating new Excalidraw content.
   * Returns the initial data structure for a new drawing.
   *
   * @param ctx - Plugin context
   * @param parentContent - The parent content item
   * @returns Initial content configuration
   */
  async onContentCreate(
    ctx: PluginContext,
    parentContent: Content
  ): Promise<CreatePluginContentInput> {
    console.log('[excalidraw] creating new drawing under:', parentContent.title);

    // Default empty drawing with white background
    const initialData: ExcalidrawData = {
      elements: [],
      appState: {
        viewBackgroundColor: '#ffffff',
        gridSize: null,
        zenModeEnabled: false,
        theme: 'light'
      },
      files: {}
    };

    return {
      title: 'New Drawing',
      contentTypeId: 'plugin:excalidraw:drawing',
      content: initialData
    };
  },

  /**
   * Called when loading Excalidraw content for rendering.
   * Transform stored data into the format expected by the renderer.
   *
   * @param ctx - Plugin context
   * @param content - The content being loaded
   * @returns Data for the renderer
   */
  async onContentLoad(ctx: PluginContext, content: Content): Promise<ExcalidrawData> {
    console.log('[excalidraw] loading drawing:', content.title);

    // Get stored data, provide defaults if missing
    const stored = content.content as ExcalidrawData | null;

    return {
      elements: stored?.elements ?? [],
      appState: stored?.appState ?? {
        viewBackgroundColor: '#ffffff',
        gridSize: null
      },
      files: stored?.files ?? {}
    };
  },

  /**
   * Called before saving Excalidraw content.
   * Transform renderer data into the format for storage.
   *
   * @param ctx - Plugin context
   * @param content - The content being saved
   * @param data - Data from the renderer
   * @returns Data for storage
   */
  async onContentSave(
    ctx: PluginContext,
    content: Content,
    data: unknown
  ): Promise<ExcalidrawData> {
    console.log('[excalidraw] saving drawing:', content.title);

    const excalidrawData = data as ExcalidrawData;

    // Clean up data before storage
    // - Remove deleted elements
    // - Strip volatile state
    const cleanElements = excalidrawData.elements.filter(
      (el) => !('isDeleted' in el && el.isDeleted)
    );

    const cleanAppState: Partial<AppState> = {
      viewBackgroundColor: excalidrawData.appState.viewBackgroundColor,
      gridSize: excalidrawData.appState.gridSize,
      zenModeEnabled: excalidrawData.appState.zenModeEnabled,
      theme: excalidrawData.appState.theme
    };

    return {
      elements: cleanElements,
      appState: cleanAppState,
      files: excalidrawData.files
    };
  },

  /**
   * Called when Excalidraw content is deleted.
   * Use for cleanup of associated resources.
   *
   * @param ctx - Plugin context
   * @param content - The content being deleted
   */
  async onContentDelete(ctx: PluginContext, content: Content): Promise<void> {
    console.log('[excalidraw] drawing deleted:', content.title);
    // Nothing to clean up for Excalidraw
    // If you stored external files, delete them here
  }
};
```

---

## Phase 6 — Build and Bundle

### Step 6.1: Build the Plugin

```bash
pnpm build
```

This creates `dist/bundle.js` containing your compiled plugin code.

### Step 6.2: Verify the Build

Check that the output exists and isn't empty:

```bash
ls -la dist/
# Should show bundle.js with a reasonable file size (likely 2-5 MB with Excalidraw)
```

> [!WARNING] If the build fails, check:
> - All imports are correct
> - TypeScript types match
> - React and Svelte are properly configured

### Step 6.3: Development Mode (Optional)

For iterative development, use watch mode:

```bash
pnpm dev
```

This rebuilds automatically when you save changes.

---

## Phase 7 — Install the Plugin

You have two options for installing your plugin:

### Option A: Using the Install Wizard (Recommended)

1. Open Palimpsest
2. Go to **Settings** → **Plugins**
3. Click **"Install Plugin"**
4. Select **"From Local Folder"**
5. Enter the path to your plugin directory:
   - Windows: `C:\Users\YourName\Documents\Palimpsest\Plugins\excalidraw`
   - macOS/Linux: `/Users/YourName/Documents/Palimpsest/Plugins/excalidraw`
6. Click **"Validate Plugin"**
7. Review the validation results
8. Click **"Install Plugin"**

### Option B: Manual Installation

If your plugin is already in the correct location (`Documents/Palimpsest/Plugins/`), simply:

1. Restart Palimpsest, or
2. Go to **Settings** → **Plugins** and wait for discovery

The plugin will be auto-discovered on startup.

### Enabling the Plugin

After installation:

1. Find your plugin in the Plugins list
2. Click the **power icon** to enable it
3. The plugin is now active!

---

## Phase 8 — Test and Debug

### Step 8.1: Create Test Content

1. Open a project
2. Right-click on any content item (e.g., a chapter)
3. Select **"Add Child"** → **"Excalidraw Drawing"**
4. A new drawing should appear with an empty canvas

### Step 8.2: Test Editing

1. Draw something on the canvas
2. Wait a moment (changes are debounced)
3. Navigate away and back — your drawing should persist

### Step 8.3: Test Read-Only Mode

1. Lock the content item (right-click → Lock)
2. The drawing should display but not be editable
3. A "locked" banner should appear

### Step 8.4: Check Browser Console

Open browser developer tools (F12) and check the console for:
- `[excalidraw] plugin enabled` — Confirms plugin loaded
- `[excalidraw] creating new drawing` — Confirms content creation
- `[excalidraw] loading drawing` — Confirms content loading
- `[excalidraw] saving drawing` — Confirms saves are working

### Common Issues

| Symptom | Likely Cause | Solution |
| ------- | ------------ | -------- |
| Plugin not discovered | Manifest invalid or missing | Check `palimpsest-plugin.json` syntax |
| Plugin loads but renderer fails | Bundle path incorrect | Verify `entryPoints.frontend` matches actual path |
| React errors in console | React version mismatch | Ensure single React version in bundle |
| Drawing doesn't save | `onupdate` not called | Check `handleChange` is wired correctly |
| Styles broken | CSS not scoped | Use `:global()` for Excalidraw styles |

---

## Complete Source Code

Here's the complete file listing for reference:

### File: `palimpsest-plugin.json`

```json
{
  "name": "excalidraw",
  "displayName": "Excalidraw",
  "version": "1.0.0",
  "palimpsestVersion": "^1.0.0",
  "description": "Add Excalidraw whiteboard drawings to your writing projects.",
  "author": {
    "name": "Your Name"
  },
  "tier": "core",
  "capabilities": {
    "contentTypes": true,
    "panels": false,
    "contextMenuItems": false,
    "jobHandlers": false,
    "restEndpoints": false
  },
  "permissions": {
    "fileSystem": false,
    "network": false,
    "localStorage": true,
    "clipboard": true
  },
  "entryPoints": {
    "frontend": "dist/bundle.js"
  },
  "contentTypes": [
    {
      "id": "drawing",
      "name": "Excalidraw Drawing",
      "icon": "PenTool",
      "color": "#6965db",
      "hasContent": true,
      "canHaveChildren": false,
      "canBeChildOf": "*",
      "renderer": "ExcalidrawRenderer"
    }
  ]
}
```

### File: `package.json`

```json
{
  "name": "palimpsest-plugin-excalidraw",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite build --watch",
    "build": "vite build"
  },
  "dependencies": {
    "@excalidraw/excalidraw": "^0.17.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@sveltejs/vite-plugin-svelte": "^4.0.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "svelte": "^5.0.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0"
  }
}
```

---

## Troubleshooting

### "Plugin not found" after installation

1. Verify the folder is in the correct location:
   ```
   Documents/Palimpsest/Plugins/excalidraw/
   ```
2. Verify `palimpsest-plugin.json` exists at the root
3. Check the manifest is valid JSON (no trailing commas, etc.)

### "Failed to load frontend module"

1. Verify `dist/bundle.js` exists
2. Check the `entryPoints.frontend` path in manifest matches
3. Rebuild with `pnpm build`

### "ExcalidrawRenderer is not defined"

1. Verify the export name in `src/index.ts` matches `renderer` in manifest
2. Check that the Svelte component is exported as default

### React "Invalid hook call" error

This usually means multiple React versions are bundled. Ensure:
1. Only one version of React in `package.json`
2. Vite isn't bundling duplicate React instances

### Drawing changes not persisting

1. Check browser console for save errors
2. Verify `onupdate` callback is being called
3. Check the debounce timeout isn't too long

---

## Next Steps

Now that you have a working plugin, consider:

1. **Adding context menu items** — Let users export drawings as PNG/SVG
2. **Adding a panel** — Show a drawing library or recent drawings
3. **Publishing** — Share your plugin with the Palimpsest community

For more advanced features, see:
- [[Palimpsest-Writing-Software/features/plugins/overview\|Plugin System Overview]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/overview\|Frontend Plugin Module]]
- [[Palimpsest-Writing-Software/api/plugins/overview\|Plugin API Reference]]

---

## Related

- [[Palimpsest-Writing-Software/features/plugins/overview\|Plugin System Overview]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/pluginLoader\|PluginLoader.ts]]
- [[Palimpsest-Writing-Software/modules/plugins-frontend/contentRendererRegistry\|ContentRendererRegistry]]
- [[Palimpsest-Writing-Software/api/plugins/overview\|Plugin API]]
