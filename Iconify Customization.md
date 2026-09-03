# Adding Iconify Icon Support to Fumadocs

This guide explains how to add Iconify icon support to a Fumadocs project while maintaining compatibility with the built-in Lucide icons plugin.

## Overview

Fumadocs comes with built-in support for Lucide icons using the `lucideIconsPlugin()`. This implementation adds Iconify support that co-exists with Lucide without conflicts by:

- Using different naming conventions (Lucide: `Building`, Iconify: `source:name`)
- Creating a wrapper for Lucide that skips Iconify-formatted icons
- Adding a separate Iconify plugin that only processes icons with colons

## Prerequisites

- A working Fumadocs project with `fumadocs-core` and `lucide-react` installed
- Package manager (npm, pnpm, or yarn)

## Implementation Steps

### Step 1: Install Iconify React

Install the `@iconify/react` package:

```bash
pnpm install @iconify/react
```

### Step 2: Create the Iconify Plugin

Create a new file at `<dir>/src/lib/iconify-plugin.tsx`:

```tsx
import { Icon } from '@iconify/react';
import { createElement } from 'react';

interface IconifyPluginOptions {
  defaultIcon?: string;
}

/**
 * Convert icon names into Iconify Icons, requires `@iconify/react` to be installed.
 *
 * Icon names should follow the format: "source:name" (e.g., "cbi:claude-clawd", "lucide:home")
 */
export function iconifyPlugin(options: IconifyPluginOptions = {}) {
  const { defaultIcon } = options;

  return {
    name: 'fumadocs:iconify-icons',
    transformPageTree: {
      file: (node: any) => {
        if (node.icon !== undefined && typeof node.icon === 'string' && node.icon.includes(':')) {
          node.icon = createElement(Icon, { icon: node.icon });
        } else if (node.icon === undefined && defaultIcon) {
          node.icon = createElement(Icon, { icon: defaultIcon });
        }
        return node;
      },
      folder: (node: any) => {
        if (node.icon !== undefined && typeof node.icon === 'string' && node.icon.includes(':')) {
          node.icon = createElement(Icon, { icon: node.icon });
        } else if (node.icon === undefined && defaultIcon) {
          node.icon = createElement(Icon, { icon: defaultIcon });
        }
        return node;
      },
      separator: (node: any) => {
        if (node.icon !== undefined && typeof node.icon === 'string' && node.icon.includes(':')) {
          node.icon = createElement(Icon, { icon: node.icon });
        } else if (node.icon === undefined && defaultIcon) {
          node.icon = createElement(Icon, { icon: defaultIcon });
        }
        return node;
      },
    },
  };
}
```

### Step 3: Create the Lucide Plugin Wrapper

Create a new file at `<dir>/src/lib/lucide-plugin-wrapper.tsx`:

This wrapper prevents the Lucide plugin from processing Iconify-formatted icons (those containing `:`).

```tsx
import { createElement } from 'react';
import { icons } from 'lucide-react';

/**
 * Custom Lucide Icons plugin that skips Iconify-formatted icons (those containing ':')
 * This allows Lucide and Iconify to co-exist without conflicts.
 */
export function lucideIconsPluginWrapper(options: { defaultIcon?: keyof typeof icons } = {}) {
  const { defaultIcon } = options;

  return {
    name: 'fumadocs:lucide-icons',
    transformPageTree: {
      file: (node: any) => {
        if (node.icon !== undefined && typeof node.icon === 'string') {
          // Skip icons with colon (Iconify format)
          if (node.icon.includes(':')) {
            return node;
          }
          const Icon = icons[node.icon as keyof typeof icons];
          if (!Icon) {
            console.warn(`[lucide-icons-plugin] Unknown icon detected: ${node.icon}.`);
            return node;
          }
          node.icon = createElement(Icon);
        } else if (node.icon === undefined && defaultIcon) {
          node.icon = createElement(icons[defaultIcon]);
        }
        return node;
      },
      folder: (node: any) => {
        if (node.icon !== undefined && typeof node.icon === 'string') {
          // Skip icons with colon (Iconify format)
          if (node.icon.includes(':')) {
            return node;
          }
          const Icon = icons[node.icon as keyof typeof icons];
          if (!Icon) {
            console.warn(`[lucide-icons-plugin] Unknown icon detected: ${node.icon}.`);
            return node;
          }
          node.icon = createElement(Icon);
        } else if (node.icon === undefined && defaultIcon) {
          node.icon = createElement(icons[defaultIcon]);
        }
        return node;
      },
      separator: (node: any) => {
        if (node.icon !== undefined && typeof node.icon === 'string') {
          // Skip icons with colon (Iconify format)
          if (node.icon.includes(':')) {
            return node;
          }
          const Icon = icons[node.icon as keyof typeof icons];
          if (!Icon) {
            console.warn(`[lucide-icons-plugin] Unknown icon detected: ${node.icon}.`);
            return node;
          }
          node.icon = createElement(Icon);
        } else if (node.icon === undefined && defaultIcon) {
          node.icon = createElement(icons[defaultIcon]);
        }
        return node;
      },
    },
  };
}
```

### Step 4: Update the Source Configuration

Modify `<dir>/src/lib/source.ts` to use both plugins:

**Before:**
```ts
import { loader } from 'fumadocs-core/source';
import { lucideIconsPlugin } from 'fumadocs-core/source/lucide-icons';
import { docsContentRoute, docsImageRoute, docsRoute } from './shared';
import { defineDocs } from 'fumadocs-mdx/macro';
import { metaSchema, pageSchema } from 'fumadocs-core/source/schema';

// ... docs definition ...

export const source = loader({
  baseUrl: docsRoute,
  source: docs.toFumadocsSource(),
  plugins: [lucideIconsPlugin()],
});
```

**After:**
```ts
import { loader } from 'fumadocs-core/source';
import { lucideIconsPluginWrapper } from './lucide-plugin-wrapper';
import { iconifyPlugin } from './iconify-plugin';
import { docsContentRoute, docsImageRoute, docsRoute } from './shared';
import { defineDocs } from 'fumadocs-mdx/macro';
import { metaSchema, pageSchema } from 'fumadocs-core/source/schema';

// ... docs definition ...

export const source = loader({
  baseUrl: docsRoute,
  source: docs.toFumadocsSource(),
  plugins: [lucideIconsPluginWrapper(), iconifyPlugin()],
});
```

## Files Created/Modified

### Files Created:
1. **`<dir>/src/lib/iconify-plugin.tsx`** - The Iconify plugin implementation
2. **`<dir>/src/lib/lucide-plugin-wrapper.tsx`** - Wrapper for Lucide that skips Iconify icons

### Files Modified:
1. **`<dir>/src/lib/source.ts`** - Updated imports and plugin configuration
2. **`<dir>/package.json`** - Updated dependencies (via package manager)

## Usage

Once implemented, you can use both icon systems in your `meta.json` files:

### Lucide Icons (Existing Format)
```json
{
  "title": "Getting Started",
  "icon": "Building"
}
```

### Iconify Icons (New Format)
```json
{
  "title": "Claude Documentation",
  "icon": "cbi:claude-clawd"
}
```

### Mixed Usage
```json
{
  "title": "Documentation",
  "pages": [
    {
      "title": "Overview",
      "icon": "Home"
    },
    {
      "title": "API Reference",
      "icon": "mdi:api"
    },
    {
      "title": "Configuration",
      "icon": "Settings"
    },
    {
      "title": "React Guide",
      "icon": "logos:react"
    }
  ]
}
```

## How It Works

### Icon Name Detection

The system automatically detects which icon library to use based on the icon name format:

- **Contains `:` (colon)** → Iconify icon (e.g., `mdi:home`, `cbi:claude-clawd`)
- **No `:` (colon)** → Lucide icon (e.g., `Building`, `Home`, `Settings`)

### Plugin Execution Order

1. **Lucide Plugin Wrapper** runs first
   - Processes icons without colons
   - Skips icons with colons (leaves them as strings)

2. **Iconify Plugin** runs second
   - Only processes string icons that contain colons
   - Converts them to Iconify React components

This ensures no conflicts between the two systems.

## Finding Iconify Icons

Browse available Iconify icons at: https://icon-sets.iconify.design/

Popular icon sets include:

- `mdi:` - Material Design Icons
- `logos:` - Brand logos  
- `heroicons:` - Heroicons
- `tabler:` - Tabler Icons
- `carbon:` - Carbon Design System
- `ph:` - Phosphor Icons
- `lucide:` - Lucide icons (via Iconify, though native Lucide is preferred)

## Troubleshooting

### Icons Not Rendering

If Iconify icons aren't rendering:

1. Verify the icon name format includes a colon (e.g., `mdi:home`, not `mdi-home`)
2. Check that `@iconify/react` is installed
3. Ensure the icon exists at https://icon-sets.iconify.design/
4. Check the browser console for errors

### Lucide Icons Breaking

If Lucide icons stop working:

1. Ensure the `lucideIconsPluginWrapper` is listed before `iconifyPlugin` in the plugins array
2. Verify icon names don't contain colons
3. Check that `lucide-react` is still installed

### Type Errors

Run type checking to catch issues:

```bash
pnpm run types:check
# or
npm run types:check
```

## Notes

- The Iconify plugin downloads icons on-demand from the Iconify CDN
- For production, consider using `@iconify/json` for offline icon data
- Both plugins support a `defaultIcon` option for fallback icons
- The wrapper approach preserves all existing Lucide functionality
