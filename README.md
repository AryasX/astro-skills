# OpenCode Astro Skills

A curated collection of OpenCode skills for Astro v6 development.

## What are these skills?

OpenCode Skills are specialized instructions that guide AI coding assistants (like OpenCode itself) with domain-specific knowledge. When a skill is loaded, the assistant gains deep, up-to-date expertise in that framework or tool — covering APIs, conventions, config, migration paths, and best practices.

This repository contains four skills for **Astro v6** and its ecosystem.

## How they were built

- Created with **OpenCode** using **DeepSeek** as the underlying model
- Cross-checked multiple times against the official Astro documentation (docs.astro.build) for accuracy and currency
- Focus on **freshness**: Astro v6, Vite 7, Zod 4, mandatory Content Layer API, `@astrojs/cloudflare` v13, Drizzle ORM on libSQL
- Legacy patterns (e.g., `src/content/config.ts`) are explicitly marked as deprecated; only current APIs are recommended

## Quick Start

Clone the repository and symlink (or copy) the skills into your OpenCode config directory:

```bash
git clone https://github.com/AryasX/astro-skills.git
mkdir -p ~/.config/opencode/skills
ln -s "$(pwd)/astro-skills/"* ~/.config/opencode/skills/
```

Alternatively, copy individual skill folders into `~/.config/opencode/skills/`. OpenCode will pick them up automatically.

To verify a skill is loaded, check:

```bash
ls ~/.config/opencode/skills/
```

## Skill Overview

| Skill | Description |
|-------|-------------|
| `astro` | Core Astro v6 – components, routing, content collections, islands architecture, SSR, actions, sessions, view transitions, and deployment. Covers React, Vue, Svelte, Solid, Preact, Alpine.js. Biases toward Astro v6 APIs, Vite 7, Zod 4. |
| `astro-cloudflare` | Deploy Astro v6 on Cloudflare Workers with `@astrojs/cloudflare` v13. Covers the workerd runtime, bindings (KV, D1, R2, DO, Queues), env access, sessions, image optimization, and migration from v12/Pages. |
| `astro-content` | Manage content collections in Astro v6 with the mandatory Content Layer API. Covers build-time loaders (glob, file), custom loaders, live collections, Zod 4 schemas, cross-references, Markdown/MDX rendering, and CMS integration. |
| `astro-db` | Build database-driven Astro apps with `@astrojs/db` and the built-in Drizzle ORM on libSQL. Covers table definitions, seeding, querying, filtering, relationships, batch transactions, Turso/libSQL deployment, and schema pushing. |

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
