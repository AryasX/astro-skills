# Astro v6 Skills

A collection of up-to-date reference guides for Astro v6 and its ecosystem — covering the full stack from content modeling and server-side rendering to database integration and Cloudflare Workers deployment.

## Version Focus

| Technology | Version |
|-----------|---------|
| Astro | 6.x (latest: 6.2) |
| Vite | 7 |
| Zod | 4.x (standalone validators, `{ error }` syntax, output-type defaults) |
| Content Layer API | Mandatory (`src/content/config.ts` removed) |
| `@astrojs/cloudflare` | 13.x (workerd runtime, `cloudflare:workers` imports) |
| Astro DB / Drizzle ORM | Built-in on libSQL |
| Node.js | >= 22.12 |

All skills were cross-checked multiple times against the official documentation at [docs.astro.build](https://docs.astro.build) for accuracy and currency. Legacy patterns (e.g., `output: 'hybrid'`, `src/content/config.ts`, `Astro.locals.runtime.env`) are explicitly marked as deprecated — only current v6 APIs are recommended.

---

## Skills

### `astro` — Core Framework

Covers the full Astro v6 surface: project setup, file-based routing (static, dynamic, catch-all), API endpoints, i18n routing, middleware, `.astro` component syntax with props, slots, and template directives. The islands architecture section documents all client directives (`client:load`, `client:idle`, `client:visible`, `client:media`, `client:only`), framework integration (React, Vue, Svelte, Solid, Preact, Alpine.js), cross-island state sharing via Nano Stores, and Server Islands with `server:defer`. Styling covers scoped/global/inline CSS, Tailwind, Sass, the Images API (`getImage`, `Image`), and the new v6 Fonts API (`fontProviders.fontsource()`, `fontProviders.google()`). SSR covers adapters, cookies, locals, CSP configuration, experimental route caching (`memoryCache()`), and streaming. Actions use `defineAction()` with Zod 4 input validation, form handling, `isInputError()`, and POST/Redirect/GET persistence. Sessions provide typed server-state with drivers per platform. View Transitions cover `<ClientRouter />`, `navigate()`, lifecycle events, and element persistence. Deployment covers static, SSR, Node/Docker, and all four adapters.

```ts
// Server-side action with Zod 4 validation
import { defineAction, ActionError } from 'astro:actions'
import { z } from 'astro/zod'

export const server = {
  subscribe: defineAction({
    accept: 'form',
    input: z.object({
      email: z.email(),
      name: z.string().min(2, { error: 'Name too short' }),
    }),
    handler: async ({ email, name }) => {
      // use Astro.cookies, session, callAction, etc.
      return { success: true }
    },
  }),
}
```

**Breaking changes in v6:** `z` imports moved to `astro/zod`, `<ClientRouter />` replaces `<ViewTransitions />`, `output: 'hybrid'` removed (use per-page `prerender = false`), event name constants removed (use string literals), Node >= 22.12 required.

---

### `astro-cloudflare` — Cloudflare Workers Deployment

Deploy Astro v6 to Cloudflare Workers with `@astrojs/cloudflare` v13. Covers adapter setup (`astro.config.mjs`), optional `wrangler.jsonc` configuration, the five image service modes (`cloudflare-binding`, `compile`, `passthrough`, hybrid, custom), `nodejs_compat` and CJS pre-compilation, and custom entrypoints with `handle()` from `@astrojs/cloudflare/handler`. All Cloudflare bindings are documented with full API: KV (`get`, `put`, `list`, `delete`), D1 (`prepare().bind().all()`), R2 (`put`, `get`, `delete`), Durable Objects (RPC + legacy fetch patterns), Queues (`send`, `sendBatch`), environment variables via `import { env } from 'cloudflare:workers'`, and the `cf` object via `Astro.request.cf`. Sessions auto-provision a KV namespace. Deployment covers build, preview on workerd, environment-specific deploys, and CI/CD with GitHub Actions.

```jsonc
// wrangler.jsonc
{
  "name": "my-astro-app",
  "main": "@astrojs/cloudflare/entrypoints/server",
  "compatibility_date": "2025-05-04",
  "compatibility_flags": ["nodejs_compat"],
  "kv_namespaces": [{ "binding": "SESSION", "id": "..." }],
  "d1_databases": [{ "binding": "DB", "database_id": "..." }]
}
```

```ts
// astro.config.mjs
import cloudflare from '@astrojs/cloudflare'

export default defineConfig({
  output: 'server',
  adapter: cloudflare({
    imageService: 'cloudflare-binding',
    sessionKVBindingName: 'SESSION',
  }),
})
```

**Migration from v12/Pages:** `Astro.locals.runtime.env` → `import { env } from 'cloudflare:workers'`, `Astro.locals.runtime.cf` → `Astro.request.cf`, `wrangler.toml` → `wrangler.jsonc` (optional), `createExports()` removed → standard `handle()` + `export default { fetch() }`, Cloudflare Pages support removed entirely.

---

### `astro-content` — Content Layer API

Manages content collections in Astro v6 using the mandatory Content Layer API. Configuration lives in `src/content.config.ts` (not `config.ts`). Covers the `glob()` loader (micromatch patterns, custom IDs), the `file()` loader (single-file multi-entry collections, CSV/TSV parsing), and custom `Loader` implementations for CMS/API integration with `store.set()`. Zod 4 schema reference covers all validators, optional/default/nullable/prefault, and cross-references via `reference('collectionName')`. Live collections use `defineLiveCollection()` with a `LiveLoader` for per-request data fetching. Querying covers `getCollection()`, `getEntry()`, `getEntries()`, filtering, sorting (non-deterministic by default), `render(entry)` returning `{ Content, headings }`, and `getStaticPaths()` integration. A migration guide covers the 7-step process from legacy `src/content/config.ts` to the Content Layer API.

```ts
// src/content.config.ts
import { defineCollection, z } from 'astro/zod'
import { glob } from 'astro/loaders'

export const collections = {
  blog: defineCollection({
    loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
    schema: z.object({
      title: z.string(),
      date: z.coerce.date(),
      draft: z.boolean().default(false),
      tags: z.array(z.string()).optional(),
      author: reference('authors'),
    }),
  }),
}
```

```astro
---
// src/pages/blog/[...slug].astro
import { getCollection, render } from 'astro:content'

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft)
  return posts.map(post => ({ params: { slug: post.id }, props: { post } }))
}

const { post } = Astro.props
const { Content } = await render(post)
---

<article>
  <h1>{post.data.title}</h1>
  <Content />
</article>
```

**Breaking changes:** Config file is `src/content.config.ts`, Zod imported from `astro/zod`, `type: 'content'` / `type: 'data'` removed (use loaders), `entry.slug` → `entry.id`, `entry.render()` → `render(entry)` (imported).

---

### `astro-db` — Database with Drizzle ORM

Builds database-driven Astro apps with `@astrojs/db` and the built-in Drizzle ORM on libSQL. Schema definition covers `defineTable()`, all column types (`text`, `number`, `boolean`, `date`, `json`), `primaryKey`, `unique`, `optional`, `default` (value or lazy function), and foreign keys with arrow-function references. Seeding uses `db/seed.ts` with batch inserts. Querying covers the full Drizzle surface: `select`/`from`/`where` with all comparison operators (`eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `like`, `ilike`, `between`, `isNull`, `inArray`), `insert`/`update`/`delete` with `returning()`, joins (`innerJoin`, `leftJoin`, `rightJoin`, `fullJoin`), aggregation (`count`, `avg`, `sum`, `max`, `min`, `groupBy`), ordering/limits, raw SQL with the `sql` template tag, and `db.batch()` for atomic multi-query transactions. Deployment covers Turso setup (CLI, URL, token, `astro db push --remote`), environment variables, all libSQL connection URL schemes (including embedded replicas with sync), Cloudflare Workers deploy, schema migrations (rename tables without data loss), and migration from Astro Studio.

```ts
// db/config.ts
import { defineDb, defineTable, column } from 'astro:db'

const User = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    email: column.text({ unique: true }),
    name: column.text(),
    createdAt: column.date({ default: () => new Date() }),
  },
})

export default defineDb({ tables: { User } })
```

```ts
// In an API endpoint or action
import { db, User } from 'astro:db'

const users = await db
  .select({ id: User.id, name: User.name })
  .from(User)
  .where(eq(User.email, email))
  .orderBy(User.name)
  .limit(10)
```

---

## How They Were Built

- Created with **OpenCode** using **DeepSeek** as the underlying model
- Inspired by Cloudflare's skill library for structure and best practices in skill authoring
- Cross-checked multiple times against the official Astro documentation for accuracy
- Every code example and API reference was verified against current Astro v6 / Vite 7 / Zod 4 behavior

## Usage

Clone and symlink into your OpenCode skills directory:

```bash
git clone https://github.com/AryasX/astro-skills.git
mkdir -p ~/.config/opencode/skills
ln -s "$(pwd)/astro-skills/"* ~/.config/opencode/skills/
```

To verify:

```bash
ls ~/.config/opencode/skills/
```

## Repository Structure

```
astro-skills/
├── astro/                  # Core Astro v6 skill
│   ├── SKILL.md
│   └── references/
├── astro-cloudflare/       # Astro + Cloudflare Workers skill
│   ├── SKILL.md
│   └── references/
├── astro-content/          # Content Layer API skill
│   ├── SKILL.md
│   └── references/
└── astro-db/               # Astro DB / Drizzle ORM skill
    ├── SKILL.md
    └── references/
```

## License

MIT
