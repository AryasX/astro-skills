# Deployment

## Static (SSG) — Default

No adapter needed.

```bash
npm run build       # Outputs to dist/
```

Deploy `dist/` to any static host: GitHub Pages, Netlify, Vercel, Cloudflare Pages, S3, etc.

```js
// astro.config.mjs
export default defineConfig({
  output: 'static',   // Default
  site: 'https://example.com',
});
```

## Server (SSR) — Requires Adapter

```bash
npm install @astrojs/cloudflare    # or @astrojs/node, @astrojs/vercel, @astrojs/netlify
```

```js
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',
  adapter: cloudflare(),
});
```

### On-demand rendering in static mode

In `output: 'static'` mode, pages can opt into on-demand rendering per-route. No separate `hybrid` output mode needed — this replaces the removed `output: 'hybrid'` from v5.

```astro
---
// Pages that need on-demand rendering:
export const prerender = false;
---
```

## Adapter Quick Reference

| Adapter | Install | For |
|---------|---------|-----|
| `@astrojs/cloudflare` | `npx astro add cloudflare` | Cloudflare Workers (v13+, workerd) |
| `@astrojs/node` | `npx astro add node` | Node.js, Express, standalone server |
| `@astrojs/vercel` | `npx astro add vercel` | Vercel (serverless + edge) |
| `@astrojs/netlify` | `npx astro add netlify` | Netlify (functions + edge) |

## Node Adapter Modes

```js
import node from '@astrojs/node';

// Standalone server (recommended)
adapter: node({ mode: 'standalone' })

// Middleware mode (Express, Connect, etc.)
adapter: node({ mode: 'middleware' })
```

With `standalone`, build produces an `entry.mjs` you can run with `node`.

## Docker Deployment

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 4321
CMD ["node", "dist/server/entry.mjs"]
```

Requires `@astrojs/node` with `mode: 'standalone'`.

## Environment Variables

```env
# .env
PUBLIC_API_URL=https://api.example.com
```

```astro
---
// Client-safe (prefix PUBLIC_):
const apiUrl = import.meta.env.PUBLIC_API_URL;

// Server-only:
const secret = import.meta.env.SECRET_KEY;
---
```

## Build Customization

```js
export default defineConfig({
  build: {
    format: 'directory',   // 'directory' (default) | 'file' | 'preserve'
    assets: '_custom',     // Assets directory name (default: '_astro')
  },
  vite: {
    build: {
      minify: false,       // Disable minification for debugging
    },
  },
});
```
