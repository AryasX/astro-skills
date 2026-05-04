---
name: astro-db
description: Build database-driven Astro apps with Astro DB and Drizzle ORM. Covers table definition, seeding, querying, filtering, relationships, batch transactions, Turso/libSQL connections, schema pushing, and integration patterns. Biases towards Drizzle ORM and Zod 4.
compatibility: opencode
metadata:
  package: "@astrojs/db"
  orm: "Drizzle ORM (built-in)"
  database: "libSQL (SQLite)"
  website: "https://docs.astro.build/en/guides/astro-db/"
references:
  - schema
  - querying
  - deployment
---

# Astro DB Skill

Database-driven Astro apps with Drizzle ORM on libSQL. Develop locally, deploy to Turso or any libSQL-compatible host.

---

## Quick Decision Table

| Task | Reference |
|------|-----------|
| Define tables and columns | `references/schema.md` |
| Set up foreign key references | `references/schema.md` |
| Seed dev data | `references/schema.md` |
| Select / query data | `references/querying.md` |
| Insert / update / delete | `references/querying.md` |
| Filter and sort | `references/querying.md` |
| Join across tables | `references/querying.md` |
| Batch transactions | `references/querying.md` |
| Raw SQL queries | `references/querying.md` |
| Connect to Turso / libSQL | `references/deployment.md` |
| Push schema to remote | `references/deployment.md` |
| Rename a table (preserving data) | `references/deployment.md` |
| Build integrations with DB | `references/schema.md` |
| Migrate from Astro Studio | `references/deployment.md` |

---

## Quick Setup

```bash
npx astro add db     # Creates db/config.ts
```

```ts
// db/config.ts
import { defineDb, defineTable, column } from 'astro:db';

const Comment = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    authorId: column.number({ references: () => Author.columns.id }),
    body: column.text(),
    createdAt: column.date({ default: new Date() }),
  },
});

export default defineDb({ tables: { Comment } });
```

Key imports: `db`, table objects, `eq`, `like`, `gt`, `count`, `sql` — all from `astro:db`.

---

## Column Type Quick Reference

```ts
column.text()        // string
column.number()      // integer
column.boolean()     // true/false
column.date()        // JavaScript Date
column.json()        // untyped JSON
```

Options: `{ primaryKey: true }`, `{ unique: true }`, `{ optional: true }`, `{ default: value }`, `{ references: () => Table.columns.id }`.

---

## Common Query Patterns

```ts
import { db, Comment, Author, eq, like, count } from 'astro:db';

// Select all
await db.select().from(Comment);

// Filter
await db.select().from(Comment).where(eq(Comment.id, 1));
await db.select().from(Comment).where(like(Comment.body, '%search%'));

// Insert
await db.insert(Comment).values({ authorId: 1, body: 'Hello' });

// Update
await db.update(Comment).set({ body: 'Updated' }).where(eq(Comment.id, 1));

// Delete
await db.delete(Comment).where(eq(Comment.id, 1));

// Join
await db.select().from(Comment).innerJoin(Author, eq(Comment.authorId, Author.id));

// Batch
await db.batch([...queries]);
```

---

## Environment Variables

```env
ASTRO_DB_REMOTE_URL=libsql://my-db-org.turso.io
ASTRO_DB_APP_TOKEN=eyJhbGciOiJF...
```

Build with remote: `astro build --remote`
Dev with remote: `astro dev --remote` (⚠️ touches production DB)

## Retrieval Sources

| Source | URL |
|--------|-----|
| Astro DB docs | `https://docs.astro.build/en/guides/astro-db/` |
| DB integration ref | `https://docs.astro.build/en/guides/integrations-guide/db/` |
| Drizzle ORM docs | `https://orm.drizzle.team/` |

---

## References

| Reference File | When to Load |
|---|---|
| `references/schema.md` | defineTable, column types/options, references, seeding, integrations |
| `references/querying.md` | Select, insert, update, delete, filter, join, batch, raw SQL |
| `references/deployment.md` | Turso setup, libSQL URLs, push, rename, migrate from Studio |
