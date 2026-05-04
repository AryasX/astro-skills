---
name: astro-cloudflare
description: Deploy Astro v6 on Cloudflare Workers with @astrojs/cloudflare v13. Covers workerd runtime, Cloudflare bindings (KV, D1, R2, DO, Queues), env access, sessions, image optimization, and migration from v12/Pages. Biases towards cloudflare:workers imports and v13 adapter APIs.
compatibility: opencode
metadata:
  adapter: "@astrojs/cloudflare >=13.0"
  runtime: "workerd"
  astro: ">=6.0"
  website: "https://docs.astro.build/en/guides/integrations-guide/cloudflare/"
references:
  - setup
  - bindings
  - sessions-on-cf
  - deployment
  - migration-v13
---

# Astro + Cloudflare Skill

Deploy Astro v6 on Cloudflare Workers. Use the decision table below to find the right reference for your task.

---

## Quick Decision Table

| Task | Reference |
|------|-----------|
| Install adapter, configure wrangler.jsonc | `references/setup.md` |
| Access KV, D1, R2, DO, Queues from code | `references/bindings.md` |
| Access env vars and secrets | `references/bindings.md` |
| Use cf object, execution context | `references/bindings.md` |
| Generate TypeScript types | `references/bindings.md` |
| Configure sessions (KV) | `references/sessions-on-cf.md` |
| Deploy to Workers | `references/deployment.md` |
| Set up CI/CD (GitHub Actions) | `references/deployment.md` |
| Image optimization options | `references/setup.md` |
| Custom entrypoint (DO, Queues) | `references/setup.md` |
| Node.js compatibility (nodejs_compat) | `references/setup.md` |
| Migrate from Pages or v12 adapter | `references/migration-v13.md` |

---

## Key v13 Changes (Astro 6)

| v12 (Astro 5) | v13 (Astro 6) |
|---|---|
| `astro dev` on Node.js | `astro dev` on `workerd` (real runtime) |
| `Astro.locals.runtime.env` | `import { env } from 'cloudflare:workers'` |
| `Astro.locals.runtime.cf` | `Astro.request.cf` |
| `Astro.locals.runtime.ctx` | `Astro.locals.cfContext` |
| `Astro.locals.runtime.caches` | global `caches` object |
| Cloudflare Pages | **Removed** — Workers only |
| `wrangler.toml` default | `wrangler.jsonc`, **optional** |
| `main: "dist/_worker.js/index.js"` | `main: "@astrojs/cloudflare/entrypoints/server"` |
| `imageService: 'compile'` (default) | `imageService: 'cloudflare-binding'` |
| `cloudflareModules: true` | Removed — natively supported |
| Prerendering on Node | `prerenderEnvironment: 'workerd'` default |
| `createExports()` pattern | Standard `export default { fetch() }` |
| `workerEntryPoint` adapter option | Custom entrypoint via `main` in wrangler |

## Adapter Options Quick Reference

```js
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  adapter: cloudflare({
    imageService: 'cloudflare-binding',        // default: cloudflare-binding
    imagesBindingName: 'IMAGES',               // default
    sessionKVBindingName: 'SESSION',            // default
    prerenderEnvironment: 'workerd',            // default; 'node' for Node.js APIs
    remoteBindings: false,                      // true to use remote bindings in dev
    persistState: true,                         // persist state between dev restarts
  }),
});
```

## Beyond this Skill

For deeper Cloudflare topics (D1 SQL API, Durable Object alarms, WebSocket hibernation, Cache API, Workers limits, etc.), load the **cloudflare** main skill.

## Retrieval Sources

| Source | URL |
|--------|-----|
| Cloudflare adapter docs | `https://docs.astro.build/en/guides/integrations-guide/cloudflare/` |
| Adapter CHANGELOG | `https://github.com/withastro/astro/blob/main/packages/integrations/cloudflare/CHANGELOG.md` |
| Cloudflare Workers docs | `https://developers.cloudflare.com/workers/` |

---

## References

| Reference File | When to Load |
|---|---|
| `references/setup.md` | Installation, wrangler.jsonc, entrypoints, imageService, nodejs_compat |
| `references/bindings.md` | KV, D1, R2, DO, Queues, env vars, cf object, cfContext, typing |
| `references/sessions-on-cf.md` | Cloudflare KV session storage, auto-provisioning, configuration |
| `references/deployment.md` | Build, preview, deploy, environments, CI/CD |
| `references/migration-v13.md` | Pages → Workers, v12 → v13 breaking changes, common fixes |
