# Astro DB Deployment

## Connect to Turso (Managed libSQL)

```bash
# 1. Install and login
turso auth login

# 2. Create a database
turso db create my-astro-db

# 3. Get connection URL
turso db show my-astro-db
# → URL: libsql://my-astro-db-org.turso.io

# 4. Create auth token
turso db tokens create my-astro-db
# → Token: eyJhbGciOiJF...

# 5. Push schema to remote
astro db push --remote
```

## Environment Variables

```env
# .env
ASTRO_DB_REMOTE_URL=libsql://my-astro-db-org.turso.io
ASTRO_DB_APP_TOKEN=eyJhbGciOiJF...
```

Set these in both local dev and your deployment platform.

## Build with Remote Connection

```json
// package.json
{
  "scripts": {
    "build": "astro build --remote"
  }
}
```

Or manually:

```bash
astro build --remote
```

### ⚠️ Dev with Remote

```bash
astro dev --remote
```

This connects to your **production database**. All changes (inserts, updates, deletes) are persisted. Use carefully.

## Connection URL Options

### URL Schemes

```
libsql://your.server.io     — Remote server (default protocol)
http://your.server.io       — Remote via HTTP
https://your.server.io      — Remote via HTTPS
ws://your.server.io         — Remote via WebSocket
wss://your.server.io        — Remote via secure WebSocket
file:path/to/file.db        — Local file
memory:                     — In-memory
```

### Local File with Encryption

```env
ASTRO_DB_REMOTE_URL=file:local.db?encryptionKey=your-key
```

### Embedded Replica (local + sync)

```env
# In-memory with sync
ASTRO_DB_REMOTE_URL=memory:?syncUrl=libsql%3A%2F%2Fyour.server.io&syncInterval=60

# File with sync
ASTRO_DB_REMOTE_URL=file:local.db?syncUrl=libsql%3A%2F%2Fyour.server.io&syncInterval=60
```

Use URL encoding for special characters in query params (e.g., `://` → `%3A%2F%2F`).

### Any libSQL Provider

Astro DB works with any server exposing the libSQL remote protocol. Just set `ASTRO_DB_REMOTE_URL` and `ASTRO_DB_APP_TOKEN`.

---

## Schema Management

### Push Schema

```bash
# Push to remote (preserves data)
astro db push --remote

# Breaking changes (DESTROYS ALL DATA)
astro db push --remote --force-reset
```

### Rename a Table Without Data Loss

```ts
// 1. Mark old table deprecated
const Comment = defineTable({
  deprecated: true,
  columns: { /* same columns */ },
});

// 2. Add new table with new name (same columns)
const Feedback = defineTable({
  columns: { /* same columns */ },
});
```

```bash
astro db push --remote     # Creates new table, marks old deprecated
```

```ts
// 3. Remove deprecated table from config
// (after migrating data and updating code)
```

```bash
astro db push --remote     # Drops old table
```

### Execute SQL Files

```bash
# Seed production data
astro db execute db/seed.ts --remote

# Run migration
astro db execute db/migrate.ts --remote
```

```ts
// db/migrate.ts
import { db, Post } from 'astro:db';

export default async function () {
  await db.update(Post)
    .set({ published: true })
    .where(eq(Post.id, 1));
}
```

---

## Deploy to Cloudflare Workers

When using `@astrojs/cloudflare`:

```bash
astro build --remote
npx wrangler deploy
```

Set env vars for the Worker:

```bash
npx wrangler secret put ASTRO_DB_REMOTE_URL
npx wrangler secret put ASTRO_DB_APP_TOKEN
```

Or in `wrangler.jsonc`:

```jsonc
{
  "vars": {
    "ASTRO_DB_REMOTE_URL": "libsql://...",
    "ASTRO_DB_APP_TOKEN": "eyJ..."
  }
}
```

## Deploy to Other Platforms

For Node.js / Vercel / Netlify, set environment variables in their respective dashboards:

- **Vercel**: Dashboard → Settings → Environment Variables
- **Netlify**: Dashboard → Site settings → Environment variables
- **Node.js**: `.env` file or platform env vars

---

## Migrate from Astro Studio

1. Export database dump from [Astro Studio](https://studio.astro.build/) → Settings → Export Database
2. Install Turso CLI: `npm install -g turso`
3. Login: `turso auth login`
4. Create DB: `turso db create [name]`
5. Get URL: `turso db show [name]` → set `ASTRO_DB_REMOTE_URL`
6. Create token: `turso db tokens create [name]` → set `ASTRO_DB_APP_TOKEN`
7. Push schema: `astro db push --remote`
8. Import data: `turso db shell [name] < ./path/to/dump.sql`
9. Once confirmed, delete project from Astro Studio

---

## Quick Reference

| Task | Command |
|------|---------|
| Create table | `defineTable({ columns: {...} })` in `db/config.ts` |
| Seed dev data | `db/seed.ts` |
| Build for prod | `astro build --remote` |
| Push schema | `astro db push --remote` |
| Reset DB | `astro db push --remote --force-reset` |
| Execute file | `astro db execute ./file.ts --remote` |
| Local DB file | `.astro/content.db` |
| Remote env var | `ASTRO_DB_REMOTE_URL` |
| Remote auth | `ASTRO_DB_APP_TOKEN` |
