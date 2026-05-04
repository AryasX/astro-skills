# Schema Definition

## Defining Tables (`db/config.ts`)

```ts
import { defineDb, defineTable, column } from 'astro:db';

const Author = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    name: column.text(),
    email: column.text({ unique: true }),
    bio: column.text({ optional: true }),
    createdAt: column.date({ default: new Date() }),
  },
});

const Post = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    authorId: column.number({ references: () => Author.columns.id }),
    title: column.text(),
    body: column.text(),
    published: column.boolean({ default: false }),
    metadata: column.json({ optional: true }),
  },
});

export default defineDb({
  tables: { Author, Post },
});
```

## Column Types

```ts
column.text()        // string
column.number()      // integer
column.boolean()     // true/false
column.date()        // JavaScript Date object
column.json()        // untyped JSON object (any valid JSON)
```

## Column Options

```ts
column.text({ primaryKey: true })     // Primary key
column.text({ unique: true })         // Unique constraint
column.text({ optional: true })       // Nullable column
column.text({ default: 'hello' })     // Default value
column.number({                       // Foreign key
  references: () => OtherTable.columns.id,
})
```

Default can be a value or a function: `column.date({ default: () => new Date() })` — runs at insert time.

## Table References (Foreign Keys)

```ts
const Comment = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    authorId: column.number({
      references: () => Author.columns.id,  // FK to Author
    }),
    postId: column.number({
      references: () => Post.columns.id,    // FK to Post
    }),
    body: column.text(),
  },
});
```

Use arrow function `() => Table.columns.id` for forward references — tables may not be defined in order.

## Seeding Development Data (`db/seed.ts`)

```ts
import { db, Author, Post, Comment } from 'astro:db';

export default async function () {
  await db.insert(Author).values([
    { id: 1, name: 'Alice', email: 'alice@example.com' },
    { id: 2, name: 'Bob', email: 'bob@example.com' },
  ]);

  await db.insert(Post).values([
    { id: 1, authorId: 1, title: 'Hello World', body: 'First post', published: true },
    { id: 2, authorId: 2, title: 'Second', body: 'Another post', published: false },
  ]);

  await db.insert(Comment).values([
    { authorId: 1, postId: 1, body: 'Great post!' },
    { authorId: 2, postId: 1, body: 'Thanks!' },
  ]);
}
```

The dev server reseeds whenever `db/seed.ts` changes. Local DB lives in `.astro/content.db`.

## Integration Setup

```ts
// my-integration/index.ts
import { defineDbIntegration } from '@astrojs/db/utils';

export default function MyIntegration() {
  return defineDbIntegration({
    name: 'my-db-integration',
    hooks: {
      'astro:db:setup': ({ extendDb }) => {
        extendDb({
          configEntrypoint: '@my-package/config',
          seedEntrypoint: '@my-package/seed',
        });
      },
    },
  });
}
```

### Type-Safe Operations in Integrations

When `astro:db` types aren't generated (integration context):

```ts
import { asDrizzleTable } from '@astrojs/db/utils';
import { db } from 'astro:db';
import { Pets } from './config';

const typeSafePets = asDrizzleTable('Pets', Pets);
await db.insert(typeSafePets).values([{ name: 'Palomita', species: 'cat' }]);
```
