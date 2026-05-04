---
name: astro-content
description: Manage content collections in Astro v6 with the Content Layer API. Covers build-time loaders (glob, file), custom loaders, live content collections, Zod 4 schemas, cross-references, rendering Markdown/MDX, and CMS integration. Biases towards Content Layer API and Zod 4 over legacy collections.
compatibility: opencode
metadata:
  astro: ">=6.0"
  zod: "4.x"
  website: "https://docs.astro.build/en/guides/content-collections/"
references:
  - build-time
  - live-collections
  - querying-rendering
  - migration
---

# Astro Content Collections Skill

Manage content in Astro v6 using the **Content Layer API** (mandatory). Legacy collections (`src/content/config.ts`) are removed — use `legacy.collectionsBackwardsCompat` only for migration.

---

## Quick Decision Table

| Task | Reference |
|------|-----------|
| Set up local Markdown/MDX collections | `references/build-time.md` |
| Load JSON/YAML/TOML data files | `references/build-time.md` |
| Define Zod schemas for type safety | `references/build-time.md` |
| Build a custom loader (CMS, API) | `references/build-time.md` |
| Create cross-references between collections | `references/build-time.md` |
| Render Markdown body content | `references/querying-rendering.md` |
| Query collections (getCollection, getEntry) | `references/querying-rendering.md` |
| Filter and sort entries | `references/querying-rendering.md` |
| Generate pages from content (getStaticPaths) | `references/querying-rendering.md` |
| Real-time / live-updating content | `references/live-collections.md` |
| CMS preview without rebuild | `references/live-collections.md` |
| Migrate from legacy collections (v4) | `references/migration.md` |

---

## V6 Key Facts

| Topic | V6 Rule |
|-------|---------|
| Config file | `src/content.config.ts` (NOT `src/content/config.ts`) |
| Zod import | `import { z } from 'astro/zod'` (NOT `astro:content` or `astro:schema`) |
| Loaders import | `import { glob, file } from 'astro/loaders'` |
| Legacy content type | Removed — must use loaders |
| `entry.slug` / `entry.render()` | Replaced by `entry.id` / `render(entry)` |
| `z.string().email()` | `z.email()` (Zod 4) |
| `z.string().min(5, { message })` | `z.string().min(5, { error })` (Zod 4) |

---

## Build-Time vs Live Collections

| Feature | Build-Time | Live |
|---------|-----------|------|
| MDX rendering | ✅ | ❌ |
| Image optimization | ✅ | ❌ |
| Data caching | ✅ (persistent store) | ❌ (per-request) |
| CMS preview | ❌ (rebuild needed) | ✅ (instant) |
| Route generation | ✅ `getStaticPaths()` | ❌ (use SSR) |
| Can coexist | ✅ | ✅ (same project) |

---

## Quick Config Template

```ts
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    pubDate: z.coerce.date(),
    draft: z.boolean().default(false),
    tags: z.array(z.string()).default([]),
  }),
});

export const collections = { blog };
```

## Retrieval Sources

| Source | URL |
|--------|-----|
| Content collections docs | `https://docs.astro.build/en/guides/content-collections/` |
| Content Loader API | `https://docs.astro.build/en/reference/content-loader-reference/` |
| Zod docs | `https://zod.dev/v4/changelog` |

---

## References

| Reference File | When to Load |
|---|---|
| `references/build-time.md` | glob/file loaders, custom loaders, Zod schemas, references |
| `references/live-collections.md` | defineLiveCollection, live loaders, querying live data |
| `references/querying-rendering.md` | getCollection, getEntry, render, filtering, sorting |
| `references/migration.md` | Legacy v4 → Content Layer API v6 migration |
