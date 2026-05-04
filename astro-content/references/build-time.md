# Build-Time Content Collections

## Configuration (`src/content.config.ts`)

```ts
import { defineCollection, reference } from 'astro:content';
import { glob, file } from 'astro/loaders';
import { z } from 'astro/zod';  // Zod 4 — NOT from 'astro:content'

const blog = defineCollection({
  loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    draft: z.boolean().default(false),
    tags: z.array(z.string()).default([]),
  }),
});

const authors = defineCollection({
  loader: glob({ pattern: '**/*.json', base: './src/data/authors' }),
  schema: z.object({
    name: z.string(),
    url: z.url(),
  }),
});

const dogs = defineCollection({
  loader: file('src/data/dogs.json'),
  schema: z.object({
    breed: z.string(),
    temperament: z.array(z.string()),
  }),
});

export const collections = { blog, authors, dogs };
```

## glob() Loader

Fetches entries from a directory of files matching a glob pattern.

```ts
loader: glob({
  pattern: '**/*.md',          // Glob pattern (micromatch)
  base: './src/content/blog',  // Base directory
})
```

Supported file types: `.md`, `.mdx`, `.markdoc`, `.json`, `.yaml`, `.yml`, `.toml`.

### Custom IDs

Override auto-generated IDs via frontmatter:

```yaml
---
title: My Post
slug: custom-id/supports/slashes
---
```

Or via `generateId`:

```ts
loader: glob({
  pattern: '**/*.json',
  base: './src/data/authors',
  generateId: ({ entry }) => entry.replace(/\.json$/, ''),
}),
```

## file() Loader

Multiple entries from a single file.

```ts
loader: file('src/data/dogs.json'),
```

Array form (each object needs `id`):
```json
[
  { "id": "poodle", "coat": "curly" }
]
```

Object form (keys become IDs):
```json
{
  "poodle": { "coat": "curly" }
}
```

### Custom Parser

```ts
import { parse as parseCsv } from 'csv-parse/sync';

loader: file('src/data/cats.csv', {
  parser: (text) => parseCsv(text, { columns: true, skipEmptyLines: true }),
}),
```

### Nested JSON

```ts
const dogs = defineCollection({
  loader: file('src/data/pets.json', {
    parser: (text) => JSON.parse(text).dogs,
  }),
});
```

## Custom Build-Time Loaders

```ts
// src/loaders/cms.ts
import type { Loader } from 'astro/loaders';

export function cmsLoader({ url, apiKey }: { url: string; apiKey: string }): Loader {
  return {
    name: 'my-cms',
    load: async ({ store, logger, meta, parseData, generateDigest }) => {
      const res = await fetch(url, {
        headers: { Authorization: `Bearer ${apiKey}` },
      });
      const items = await res.json();

      for (const item of items) {
        const data = await parseData({
          id: item.slug,
          data: { title: item.title, body: item.content },
        });
        store.set({
          id: item.slug,
          data,
          digest: generateDigest(data),
          ...(item.deleted ? { deleted: true } : {}),
        });
      }
    },
  };
}
```

```ts
// src/content.config.ts
import { cmsLoader } from './loaders/cms';

const posts = defineCollection({
  loader: cmsLoader({ url: 'https://cms.example.com/api/posts', apiKey: '...' }),
});
```

Community loaders: https://astro.build/integrations/?categories%5B%5D=loaders

## Zod 4 Schemas

**Import from `astro/zod`** — NOT `astro:content` or `astro:schema` (both removed).

### Common Validators

```ts
z.string()            z.number()              z.boolean()
z.date()              z.coerce.date()         z.url()
z.email()             z.array(z.string())     z.object({})
z.enum(['a', 'b'])    z.literal('exact')      z.optional()
z.default(value)      z.nullable()            z.prefault(value)
```

### Zod 4 Changes

| Zod 3 | Zod 4 |
|-------|-------|
| `z.string().email()` | `z.email()` |
| `z.string().url()` | `z.url()` |
| `z.string().min(5, { message: '...' })` | `z.string().min(5, { error: '...' })` |
| `z.string().transform(Number).default('0')` | `z.string().transform(Number).default(0)` |
| Default matches input type | Default matches output type |

Use `.prefault()` for old behavior where defaults are parsed through transforms.

## Cross-References

```ts
import { reference } from 'astro:content';

const blog = defineCollection({
  schema: z.object({
    title: z.string(),
    author: reference('authors'),                // Single reference
    relatedPosts: z.array(reference('blog')),    // Array of references
  }),
});
```

Resolve references when querying:
```ts
const post = await getEntry('blog', 'my-post');
const author = await getEntry(post.data.author);          // Single
const related = await getEntries(post.data.relatedPosts); // Array
```

## JSON Schema Editor Support

Astro auto-generates `.astro/types.d.ts` for editor autocomplete. Restart dev or run `astro sync` if schemas feel stale.
