---
name: astro
description: Build content-driven websites with Astro v6 — components, routing, content collections, islands architecture, SSR, actions, sessions, view transitions, and deployment. Covers React, Vue, Svelte, Solid, Preact, Alpine.js. Biases towards Astro v6 APIs, Vite 7, Zod 4.
compatibility: opencode
metadata:
  version: "6.x (latest: 6.2)"
  node: ">=22.12"
  website: "https://astro.build"
references:
  - project-setup
  - routing
  - components
  - islands
  - styling-and-assets
  - server-rendering
  - actions
  - sessions
  - view-transitions
  - deployment
---

# Astro v6 Skill

Comprehensive skill for building with Astro. Use decision trees below to find the right approach, then load the referenced detail file.

Your knowledge of Astro APIs may be outdated — **prefer retrieval from Astro docs over pre-training**. Import paths, configuration, and Zod APIs changed significantly in v6.

## Retrieval Sources

| Source | URL | Use for |
|--------|-----|---------|
| Astro docs | `https://docs.astro.build/` | Configuration, API signatures, guides |
| Astro changelog | `https://github.com/withastro/astro/releases` | Recent changes, deprecations |
| Zod docs | `https://zod.dev/v4/changelog` | Zod 4 breaking changes |
| Vite docs | `https://vite.dev/guide/migration` | Vite 7 migration |

When in doubt, **trust the docs**.

---

## Decision Trees

### "I need to create pages / routes"

```
Need routing?
├─ Static content pages → .astro files in src/pages/ → references/routing.md
├─ Dynamic [slug] routes → getStaticPaths() → references/routing.md
├─ API endpoints → src/pages/api/*.ts → references/routing.md
├─ Redirects → Astro.redirect() or astro config → references/routing.md
├─ i18n routes → configure i18n in astro.config.mjs → references/routing.md
├─ Middleware → src/middleware.ts → references/routing.md
└─ SPA navigation → View Transitions → references/view-transitions.md
```

### "I need to build UI"

```
Need UI?
├─ .astro components → src/components/*.astro → references/components.md
├─ Interactive / client components → framework + client:* directive → references/islands.md
│  ├─ React → @astrojs/react
│  ├─ Vue → @astrojs/vue
│  ├─ Svelte → @astrojs/svelte
│  ├─ Solid → @astrojs/solid-js
│  ├─ Preact → @astrojs/preact
│  └─ Alpine.js → @astrojs/alpinejs
├─ Layouts (shared structure) → src/layouts/*.astro → references/components.md
├─ CSS / Sass → <style> or imports → references/styling-and-assets.md
├─ Images → <Image />, getImage() → references/styling-and-assets.md
└─ Fonts → Font component + fontProviders → references/styling-and-assets.md
```

### "I need server-side features"

```
Need SSR / server?
├─ On-demand rendering → output: 'server' + adapter → references/server-rendering.md
├─ Forms → Astro Actions (recommended) → references/actions.md
├─ Sessions → Astro.session.get()/.set() → references/sessions.md
├─ Auth / gating → middleware or action handler → references/actions.md
├─ CSP → security: { csp: true } in config → references/server-rendering.md
├─ Route caching → Astro.cache.set() (experimental) → references/server-rendering.md
└─ Server Islands → server:defer → references/islands.md
```

### "I need content management"

```
Need content?
├─ Local Markdown/MDX → glob() loader + Content Layer API → astro-content skill
├─ Remote CMS → custom loader → astro-content skill
├─ Real-time content → defineLiveCollection() → astro-content skill
├─ Database content → Astro DB / Drizzle ORM → astro-db skill
└─ Render body → render(entry) → astro-content skill
```

### "I need to deploy"

```
Need deployment?
├─ Static (SSG) → output: 'static' → references/deployment.md
├─ Cloudflare Workers → @astrojs/cloudflare v13+ → astro-cloudflare skill
├─ Netlify → @astrojs/netlify → references/deployment.md
├─ Vercel → @astrojs/vercel → references/deployment.md
├─ Node.js → @astrojs/node → references/deployment.md
└─ Docker → @astrojs/node + Dockerfile → references/deployment.md
```

---

## Quick Reference: V6 Breaking Changes

| v5 API (deprecated/removed) | v6 Replacement |
|---|---|
| `import { z } from 'astro:content'` | `import { z } from 'astro/zod'` |
| `import { z } from 'astro:schema'` | `import { z } from 'astro/zod'` |
| `<ViewTransitions />` | `<ClientRouter />` from `astro:transitions` |
| `Astro.glob()` | `import.meta.glob()` |
| `src/content/config.ts` (legacy) | `src/content.config.ts` (Content Layer API) |
| `z.string().email()` | `z.email()` (Zod 4) |
| `z.string().min(5, { message })` | `z.string().min(5, { error })` (Zod 4) |
| session driver string signature | `sessionDrivers.redis()` object API |
| Node 18/20 | Node >= 22.12 |
| Vite 6 | Vite 7 |
| `Astro.locals.runtime.env` (Cloudflare) | `import { env } from 'cloudflare:workers'` |
| `{ default: '0' }` after transform | `{ default: 0 }` matches output type (Zod 4) |

---

## References

Load these for detailed information, code examples, and edge cases:

| Reference File | When to Load |
|---|---|
| `references/project-setup.md` | Creating new project, configuration, editor setup |
| `references/routing.md` | Pages, dynamic routes, endpoints, middleware, i18n |
| `references/components.md` | .astro syntax, props, slots, layout components |
| `references/islands.md` | Client directives, hydration, framework components |
| `references/styling-and-assets.md` | CSS, images, fonts API, responsive images |
| `references/server-rendering.md` | SSR, on-demand rendering, adapters overview, CSP, route caching |
| `references/actions.md` | Define, call, forms, error handling, security |
| `references/sessions.md` | Configuration, get/set, typed sessions, drivers |
| `references/view-transitions.md` | ClientRouter, SPA navigation, transition events |
| `references/deployment.md` | Static vs SSR, adapter installation, build commands |
