# Project Setup

## Create a new project

```bash
npm create astro@latest
```

## Version requirements (v6)

- **Node.js** >= 22.12.0 (18 and 20 dropped)
- **Vite 7** (bundled with Astro)
- **Zod 4** (bundled, import from `astro/zod`)

## Project structure

```
src/
├── pages/           # REQUIRED — routes and pages
│   ├── index.astro
│   ├── about.astro
│   ├── blog/
│   │   ├── [slug].astro
│   │   └── index.astro
│   └── api/
│       └── search.ts
├── components/      # Reusable .astro and framework components
├── layouts/         # Shared layout wrappers
├── content/         # Markdown/MDX content (referenced by loaders)
├── styles/          # Global CSS
├── actions/
│   └── index.ts     # Astro Actions
├── middleware.ts     # Request middleware
├── content.config.ts # Content collections config
└── live.config.ts   # Live content collections (v6)
public/              # Static assets (copied as-is)
astro.config.mjs     # Astro configuration
tsconfig.json        # TypeScript configuration
```

## Configuration (astro.config.mjs)

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  output: 'static',          // 'static' | 'server'
  site: 'https://example.com',

  integrations: [],           // React, Vue, Svelte, MDX, etc.

  image: {
    service: {
      entrypoint: 'astro/assets/services/sharp',
    },
  },

  security: {
    csp: true,               // Content Security Policy (stable in v6)
  },

  session: {
    driver: sessionDrivers.redis({ url: process.env.REDIS_URL }),
    cookie: { secure: true },
    ttl: 3600,
  },

  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'fr', 'es'],
    routing: {
      prefixDefaultLocale: false,
      redirectToDefaultLocale: false,  // v6 default changed
    },
  },

  experimental: {
    cache: { provider: memoryCache() },     // Route caching
    queuedRendering: { enabled: true },     // ~2x faster rendering
    svgOptimizer: svgoOptimizer(),          // SVG optimization
    rustCompiler: true,                     // Faster .astro compiler
  },
});
```

## Add integrations

```bash
npx astro add react
npx astro add tailwind
npx astro add cloudflare
npx astro add db
npx astro add mdx
npx astro add sitemap
```

## TypeScript

Astro provides config templates: `astro/tsconfigs/strict`, `astro/tsconfigs/strictest`, `astro/tsconfigs/base`.

```json
// tsconfig.json
{
  "extends": "astro/tsconfigs/strict"
}
```

For content collections with the `base` template, add:

```json
{
  "compilerOptions": {
    "strictNullChecks": true,
    "allowJs": true
  }
}
```

## Editor setup

- **VS Code**: Install "Astro" extension
- **LSP**: Configured automatically via `astro.config.mjs`
- **Formatting**: Prettier with `prettier-plugin-astro`
