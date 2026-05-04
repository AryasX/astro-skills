# Routing

## File-based routing

```
src/pages/index.astro           → /
src/pages/about.astro           → /about
src/pages/blog/index.astro      → /blog
src/pages/blog/[slug].astro     → /blog/:slug (dynamic)
src/pages/[...rest].astro       → /anything/* (catch-all, rest params)
```

## Page files

```astro
---
// src/pages/about.astro
// Frontmatter: runs at build time (static) or on server (SSR)
const title = 'About Us';
---

<html>
  <body>
    <h1>{title}</h1>
  </body>
</html>
```

## Dynamic routes (getStaticPaths)

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
---
<!-- IMPORTANT: Do NOT use Astro.site inside getStaticPaths() — deprecated in v6 -->
<!-- Use import.meta.env.SITE instead -->
<h1>{post.data.title}</h1>
```

## API Endpoints

```ts
// src/pages/api/search.ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = async ({ request, url }) => {
  const query = url.searchParams.get('q');
  return new Response(JSON.stringify({ query }), {
    headers: { 'Content-Type': 'application/json' },
  });
};

export const POST: APIRoute = async ({ request }) => {
  const body = await request.json();
  return new Response(JSON.stringify({ ok: true }), { status: 201 });
};

export const DELETE: APIRoute = async ({ params }) => {
  await deleteItem(params.id);
  return new Response(null, { status: 204 });
};

export const prerender = false; // Required for on-demand rendered endpoints
```

## Redirects

```astro
---
// In component frontmatter
return Astro.redirect('/new-url');
---
```

Or in config:

```js
export default defineConfig({
  redirects: {
    '/old': '/new',
    '/blog/[...slug]': '/articles/[...slug]',
  },
});
```

## Middleware

```ts
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  // Runs before every request
  // Access: context.request, context.cookies, context.locals
  const response = await next();
  response.headers.set('X-Custom', 'value');
  return response;
});
```

### Auth gating in middleware

```ts
export const onRequest = defineMiddleware(async (context, next) => {
  if (!context.cookies.has('session')) {
    return new Response('Unauthorized', { status: 401 });
  }
  return next();
});
```

## i18n Routing

```js
// astro.config.mjs
export default defineConfig({
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'fr', 'es'],
    routing: {
      prefixDefaultLocale: false,    // /fr/about vs /en/about vs /about
      redirectToDefaultLocale: false, // v6 default: false
      fallbackType: 'redirect',      // 'redirect' | 'rewrite'
    },
  },
});
```

Access locale in components:
```astro
---
const { locale } = Astro;  // 'en' | 'fr' | 'es'
---
```

Import i18n utilities:
```ts
import { getLocaleByPath, getRelativeLocaleUrl } from 'astro:i18n';
```

## Route Priority / On-demand rendering

```astro
---
// Opt IN to on-demand rendering (in output: 'static' mode):
export const prerender = false;
---
```
