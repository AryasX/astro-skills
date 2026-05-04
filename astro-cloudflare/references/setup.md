# Cloudflare Setup

## Installation

```bash
npx astro add cloudflare
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',  // or 'hybrid'
  adapter: cloudflare(),
});
```

## Wrangler Configuration

The `wrangler.jsonc` is **optional** in v13. Astro generates defaults for basic projects. Delete it if it only contained basic settings.

### Minimal (when you need bindings)

```jsonc
{
  "main": "@astrojs/cloudflare/entrypoints/server",
  "name": "my-astro-app",
  "compatibility_date": "2026-04-01",
  "compatibility_flags": ["nodejs_compat"],
  "kv_namespaces": [
    { "binding": "MY_KV", "id": "<namespace_id>" }
  ],
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<db_id>" }
  ],
  "r2_buckets": [
    { "binding": "MY_BUCKET", "bucket_name": "my-bucket" }
  ],
  "durable_objects": {
    "bindings": [
      { "name": "MY_DO", "class_name": "MyDurableObject" }
    ]
  }
}
```

### Node.js Compatibility

For `node:buffer`, `node:crypto`, `node:path`, etc.:

```jsonc
{
  "compatibility_flags": ["nodejs_compat"]
}
```

Some Node.js APIs require polyfills (needs `nodejs_compat` + compatibility date >= 2024-09-23).

## Adapter Options (Full Reference)

```js
adapter: cloudflare({
  // Image optimization
  imageService: 'cloudflare-binding',        // default — auto-provisioned Images binding
  // imageService: 'compile',                // build-time only (prerendered routes)
  // imageService: 'passthrough',            // no optimization
  // imageService: { build: 'compile', runtime: 'cloudflare-binding' },  // hybrid
  imagesBindingName: 'IMAGES',               // default binding name

  // Sessions
  sessionKVBindingName: 'SESSION',           // default KV binding name

  // Prerendering runtime
  prerenderEnvironment: 'workerd',           // default — 'node' if Node.js APIs needed

  // Dev options
  configPath: './wrangler.jsonc',            // custom wrangler config path
  remoteBindings: false,                     // use remote bindings in dev
  persistState: true,                        // persist DO/KV state between dev restarts
  inspectorPort: 9229,                       // Chrome DevTools port

  // Advanced
  auxiliaryWorkers: [],                      // additional Workers to run in dev
}),
```

### imageService Options

| Value | Behavior |
|-------|---------|
| `'cloudflare-binding'` | Cloudflare Images binding (default in v13, auto-provisioned) |
| `'compile'` | Build-time transform (prerendered only, was default in v12) |
| `'passthrough'` | No optimization |
| `{ build: 'compile', runtime: 'cloudflare-binding' }` | Hybrid approach |
| `'custom'` | Uses your configured image service directly |

### prerenderEnvironment

- `'workerd'` (default): Prerendered pages built in workerd — matches production
- `'node'`: Use if prerendered pages need `node:fs`, `node:path`, or Node-only packages

On-demand rendered pages always use `workerd`.

## Custom Entrypoint (Durable Objects, Queues)

```jsonc
// wrangler.jsonc
{
  "main": "./src/worker.ts"
}
```

```ts
// src/worker.ts
import { handle } from '@astrojs/cloudflare/handler';
import { DurableObject } from 'cloudflare:workers';

export class MyDurableObject extends DurableObject<Env> {
  async fetch(request: Request): Promise<Response> {
    return new Response('Hello from DO');
  }
}

export default {
  async fetch(request, env, ctx) {
    // Custom pre-processing
    await env.MY_QUEUE.send({ type: 'log' });
    // Delegate to Astro
    return handle(request, env, ctx);
  },
  async queue(batch, _env) {
    console.log(`consumed: ${JSON.stringify(batch.messages)}`);
  },
} satisfies ExportedHandler<Env>;
```

Do NOT import manifest or use `createExports()` — both removed in v13.

## Pre-compile CJS Dependencies

If a dependency fails in workerd (no CommonJS support):

```js
// vite.config.ts
function optimizeDepsPlugin() {
  return {
    name: 'optimize-dependencies',
    configEnvironment(environment) {
      if (environment !== 'client') {
        return {
          optimizeDeps: { include: ['some-cjs-lib'] },
        };
      }
    },
  };
}
```

## Static Assets

### Headers (`public/_headers`)

```
/fonts/*
  Cache-Control: public, max-age=31536000, immutable
```

### Redirects (`public/_redirects`)

```
/old-page /new-page 301
```

Only for static assets. Use `Astro.redirect()` or config redirects for dynamic routes.

## Module Imports (wasm, binary, text)

```ts
import mod from './add.wasm';     // WebAssembly.Module
import data from './data.bin';     // ArrayBuffer
import text from './content.txt';   // string
```

No special configuration needed (old `cloudflareModules` option removed).
