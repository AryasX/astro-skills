# Migration: Pages → Workers / v12 → v13

## v12 → v13 Breaking Changes (Complete)

### Env Access

```ts
// v12 (REMOVED):
const { env } = Astro.locals.runtime;
const value = env.MY_KV;

// v13:
import { env } from 'cloudflare:workers';
const value = env.MY_KV;
```

### cf Object

```ts
// v12:
const { cf } = Astro.locals.runtime;

// v13:
const cf = Astro.request.cf;
```

### Execution Context

```ts
// v12:
const { ctx } = Astro.locals.runtime;
ctx.waitUntil(promise);

// v13:
const ctx = Astro.locals.cfContext;
ctx.waitUntil(promise);
```

### Caches

```ts
// v12:
const { caches } = Astro.locals.runtime;
caches.default.put(request, response);

// v13:
// global caches is available directly
caches.default.put(request, response);
```

### Entrypoint

```jsonc
// v12 (wrangler.toml):
// main = "dist/_worker.js/index.js"

// v13 (wrangler.jsonc):
{
  "main": "@astrojs/cloudflare/entrypoints/server"
}
```

### Wrangler Config

```toml
# v12: wrangler.toml
main = "dist/_worker.js/index.js"
compatibility_date = "2025-01-01"
```

```jsonc
// v13: wrangler.jsonc (optional!)
{
  // Optional — Astro generates defaults
}
```

### Custom Entrypoint (workerEntryPoint)

```js
// v12:
adapter: cloudflare({
  workerEntryPoint: {
    path: 'src/worker.ts',
    namedExports: ['MyDurableObject'],
  },
})
```

```jsonc
// v13: wrangler.jsonc
{
  "main": "./src/worker.ts"
}
```

```ts
// v12:
import { createExports } from '@astrojs/cloudflare';

// v13:
import { handle } from '@astrojs/cloudflare/handler';

export default {
  fetch(request, env, ctx) {
    return handle(request, env, ctx);
  },
};
```

### Image Service

```js
// v12 default:
adapter: cloudflare({ imageService: 'compile' })

// v13 default:
adapter: cloudflare({ imageService: 'cloudflare-binding' })
```

To keep v12 behavior:
```js
adapter: cloudflare({ imageService: 'compile' })
```

### cloudflareModules

```js
// v12:
adapter: cloudflare({ cloudflareModules: true })
// v13:
// Removed — just import .sql/.wasm/.bin/.txt directly
```

### Prerendering

```js
// v13 new option:
adapter: cloudflare({ prerenderEnvironment: 'node' })
// Use if prerendered pages need Node.js APIs (node:fs, etc.)
// Default: 'workerd'
```

---

## Cloudflare Pages → Workers Migration

### Why

Cloudflare Pages support was **removed** from `@astrojs/cloudflare` v13. Workers provides better SSR support, the real `workerd` runtime, and continues to receive updates.

### Steps

1. **Remove Pages-specific configuration** from your project
2. **Install v13 adapter**:
   ```bash
   npm install @astrojs/cloudflare@latest
   ```
3. **Create `wrangler.jsonc`** (minimal):
   ```jsonc
   { "name": "my-astro-app" }
   ```
4. **Set `output: 'server'`** in `astro.config.mjs`
5. **Update deploy command**: Use `npx wrangler deploy` instead of Pages Git integration
6. **Check for `Astro.locals.runtime` usage** — replace with new APIs (see above)

### CI/CD Migration

- **Before**: Push to Git, Cloudflare Pages builds and deploys
- **After**: GitHub Actions → `astro build` → `wrangler deploy`

### Official Cloudflare Migration Guide

https://developers.cloudflare.com/workers/static-assets/migration-guides/migrate-from-pages/

---

## Common Migration Issues

### "require is not defined" in dev

Some dependencies use CommonJS which workerd doesn't support. Create `vite.config.ts`:

```ts
export default {
  plugins: [
    {
      name: 'optimize-deps',
      configEnvironment(env) {
        if (env !== 'client') {
          return { optimizeDeps: { include: ['problematic-lib'] } };
        }
      },
    },
  ],
};
```

### "env is not defined"

You're still using `Astro.locals.runtime.env`. Switch to:
```ts
import { env } from 'cloudflare:workers';
```

### "Astro.locals.runtime is undefined"

The `runtime` object was removed from `Astro.locals` entirely. See migration table above for each property's replacement.

### Build fails with prerendering

If prerendered pages use Node.js APIs:
```js
adapter: cloudflare({ prerenderEnvironment: 'node' })
```
