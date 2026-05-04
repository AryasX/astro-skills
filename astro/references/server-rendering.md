# Server-Side Rendering (SSR)

## Output Modes

```js
// astro.config.mjs
export default defineConfig({
  output: 'static',   // Default — all pages prerendered
  output: 'server',   // All pages SSR
});
```

## Per-page SSR opt-in

In `output: 'static'` mode, individual pages can opt into on-demand rendering:

```astro
---
export const prerender = false;
---
```

## Adapters

Required for on-demand rendering (`output: 'server'` or per-page opt-out):

```bash
npx astro add cloudflare   # @astrojs/cloudflare v13+
npx astro add node         # @astrojs/node
npx astro add vercel       # @astrojs/vercel
npx astro add netlify      # @astrojs/netlify
```

```js
import cloudflare from '@astrojs/cloudflare';
export default defineConfig({
  output: 'server',
  adapter: cloudflare(),
});
```

## Defer static page building (SSR fallback)

For large sites (10,000+ pages), use SSR to defer building until first request:

```astro
---
// src/pages/catalog/[id].astro
export const prerender = false;

const { id } = Astro.params;
const product = await db.select().from(Product).where(eq(Product.id, id));
if (!product) return Astro.redirect('/404');
---
```

## Astro.request in SSR

```astro
---
// Available only in on-demand rendered pages
const url = Astro.request.url;
const headers = Astro.request.headers;
const method = Astro.request.method;
const formData = await Astro.request.formData();
---
```

## Cookies

```astro
---
// Set
Astro.cookies.set('key', 'value', { httpOnly: true, secure: true });
// Get
const value = Astro.cookies.get('key')?.value;
// Delete
Astro.cookies.delete('key', { path: '/' });
---
```

## `Astro.locals`

Pass data from middleware to pages:

```ts
// src/middleware.ts
export const onRequest = defineMiddleware(async (context, next) => {
  context.locals.user = await authenticate(context.request);
  return next();
});
```

```astro
---
// In page
const user = Astro.locals.user;
---
```

Type `Astro.locals`:

```ts
// src/env.d.ts
declare namespace App {
  interface Locals {
    user?: { id: string; name: string };
  }
}
```

---

## Content Security Policy (v6 stable)

```js
export default defineConfig({
  security: {
    csp: true,  // Auto-hashes all scripts/styles
  },
});
```

Advanced configuration:

```js
security: {
  csp: {
    algorithm: 'SHA-512',
    directives: [
      "default-src 'self'",
      "img-src 'self' https://images.cdn.example.com",
    ],
    styleDirective: { hashes: ['sha384-styleHash'] },
    scriptDirective: { hashes: ['sha384-scriptHash'] },
  },
}
```

Works with responsive images out of the box in v6.

---

## Route Caching (Experimental)

```js
// astro.config.mjs
import { memoryCache } from 'astro/config';

export default defineConfig({
  experimental: {
    cache: { provider: memoryCache() },
  },
});
```

```astro
---
// In page
Astro.cache.set({
  maxAge: 120,           // Cache for 2 minutes
  swr: 60,               // Stale-while-revalidate for 1 minute
  tags: ['home'],         // For targeted invalidation
});
---
```

Integration with content collections:
```astro
---
import { getEntry } from 'astro:content';
const product = await getEntry('products', Astro.params.slug);
// Auto-invalidates when product content changes:
Astro.cache.set(product);
---
```

---

## Streaming

Use `Response` with a `ReadableStream` for progressive rendering:

```ts
// src/pages/api/stream.ts
export const GET: APIRoute = () => {
  const stream = new ReadableStream({
    start(controller) {
      controller.enqueue(new TextEncoder().encode('chunk 1\n'));
      controller.enqueue(new TextEncoder().encode('chunk 2\n'));
      controller.close();
    },
  });
  return new Response(stream, {
    headers: { 'Content-Type': 'text/plain' },
  });
};
```
