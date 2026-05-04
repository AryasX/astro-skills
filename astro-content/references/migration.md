# Migration: Legacy → Content Layer API (v6)

## Overview

Legacy content collections (v4 style) are **removed** in v6. All collections must use the Content Layer API.

## Before → After

```ts
// BEFORE: src/content/config.ts (v4 legacy)
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content',           // ← removed
  schema: z.object({
    title: z.string(),
  }),
});

export const collections = { blog };
```

```ts
// AFTER: src/content.config.ts (v6)
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';    // NOT from 'astro:content'

const blog = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/blog' }),
  schema: z.object({ title: z.string() }),
});

export const collections = { blog };
```

## Key Changes

| Legacy (v4) | Content Layer API (v6) |
|---|---|
| `src/content/config.ts` | `src/content.config.ts` |
| `z` from `astro:content` | `z` from `astro/zod` |
| `type: 'content'` | `glob()` loader |
| `type: 'data'` | `file()` or `glob()` loader |
| `entry.slug` | `entry.id` |
| `entry.render()` | `render(entry)` from `astro:content` |
| Frontmatter-only | Zod 4 schemas |
| Automatic directory matching | Explicit loader `pattern` + `base` |

## Temporary Migration Flag

If you can't migrate immediately:

```js
// astro.config.mjs
export default defineConfig({
  legacy: {
    collectionsBackwardsCompat: true,
  },
});
```

This preserves:
- Legacy `src/content/config.ts` file path
- `type: 'content'` and `type: 'data'` without loaders
- `entry.slug` and `entry.render()` APIs
- Path-based entry IDs instead of slug-based

**This is a temporary helper.** Migrate to Content Layer API and remove this flag.

## Migration Steps

1. Rename `src/content/config.ts` → `src/content.config.ts`
2. Replace `z` import: `astro:content` → `astro/zod`
3. Remove `type: 'content'` / `type: 'data'` from each collection
4. Add `loader` to each collection:
   - Files in a directory → `glob({ pattern: '**/*.{md,mdx}', base: './src/content/<name>' })`
   - Single file → `file('src/data/<name>.json')`
5. Replace `entry.slug` → `entry.id` in templates
6. Replace `entry.render()` → `render(entry)` (imported from `astro:content`)
7. Remove `legacy.collectionsBackwardsCompat` flag
