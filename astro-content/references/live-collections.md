# Live Content Collections

Fetch content at request time — no rebuild needed.

## Configuration (`src/live.config.ts`)

```ts
import { defineLiveCollection } from 'astro:content';
import { z } from 'astro/zod';

const updates = defineLiveCollection({
  loader: myLiveLoader({ apiKey: process.env.API_KEY }),
  schema: z.object({
    slug: z.string(),
    title: z.string(),
    excerpt: z.string(),
    publishedAt: z.coerce.date(),
  }),
});

export const collections = { updates };
```

## Querying Live Data

```astro
---
import { getLiveEntry, getLiveCollection } from 'astro:content';

// Single entry
const { entry, error } = await getLiveEntry('updates', Astro.params.slug);
if (error || !entry) {
  return Astro.redirect('/404');
}

// All entries
const result = await getLiveCollection('updates');
if (result.error) {
  console.error(result.error);
}
const entries = result.entries;
---

<h1>{entry.data.title}</h1>
<p>{entry.data.excerpt}</p>
```

## Building a Live Loader

```ts
import type { LiveLoader } from 'astro:content';

function myLiveLoader({ apiKey }: { apiKey: string }): LiveLoader {
  return {
    name: 'my-live-loader',
    load: async (params) => {
      // params.context: the entry ID being queried
      const res = await fetch(
        `https://api.example.com/items/${params.context}`,
        { headers: { Authorization: `Bearer ${apiKey}` } }
      );
      if (!res.ok) {
        throw new Error(`Failed to fetch: ${res.statusText}`);
      }
      const data = await res.json();
      return {
        slug: data.slug,
        title: data.title,
        excerpt: data.excerpt,
        publishedAt: data.published_at,
      };
    },
  };
}
```

## Live vs Build-Time Comparison

| Feature | Build-Time | Live |
|---------|-----------|------|
| MDX rendering | ✅ | ❌ |
| Image optimization | ✅ | ❌ |
| Data caching | ✅ (persistent store) | ❌ (per-request) |
| Performance | ✅ (prerendered at build) | ⚠️ (fetched per request) |
| CMS preview | ❌ (needs rebuild) | ✅ (updates instantly) |
| `getStaticPaths()` | ✅ | ❌ (use SSR + dynamic routes) |
| Filter queries client-side | ✅ (filter on returned array) | ❌ (must filter in loader params) |
| Coexists with build-time | ✅ | ✅ (same project) |

**Use build-time** unless you specifically need real-time freshness.

## Error Handling

Live collections can fail at request time — always handle errors:

```astro
---
const { entry, error } = await getLiveEntry('products', Astro.params.slug);

if (error) {
  // Log the error, show fallback UI
  console.error('Failed to fetch product:', error);
  return new Response('Temporarily unavailable', { status: 503 });
}

if (!entry) {
  return Astro.redirect('/404');
}
---
```

## With Route Caching (Experimental)

Combine live collections with route caching for better performance:

```js
// astro.config.mjs
import { memoryCache } from 'astro/config';

export default defineConfig({
  experimental: { cache: { provider: memoryCache() } },
});
```

```astro
---
import { getLiveEntry } from 'astro:content';

const { entry } = await getLiveEntry('products', Astro.params.slug);

// Cache for 1 minute
Astro.cache.set({ maxAge: 60, tags: ['product', entry.id] });
---
```
