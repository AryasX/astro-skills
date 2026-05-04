# Querying with Drizzle ORM

All imports from `astro:db`:

```ts
import { db, Author, Post, Comment, eq, like, gt, and, or, count, sql } from 'astro:db';
```

---

## Select

```ts
// All rows
const posts = await db.select().from(Post);

// With columns
const result = await db.select({
  title: Post.title,
  authorId: Post.authorId,
}).from(Post);
```

Result is fully typed based on your table schema.

## Insert

```ts
// Single
await db.insert(Post).values({
  authorId: 1, title: 'New Post', body: 'Content here',
});

// Multiple
await db.insert(Post).values([
  { authorId: 1, title: 'Post 1', body: '...' },
  { authorId: 1, title: 'Post 2', body: '...' },
]);

// With returning (get inserted rows)
const result = await db.insert(Post)
  .values({ authorId: 1, title: 'New', body: '...' })
  .returning();
console.log(result[0].id); // Auto-generated ID
```

## Update

```ts
await db.update(Post)
  .set({ published: true, title: 'Updated Title' })
  .where(eq(Post.id, 1));

// With returning
const updated = await db.update(Post)
  .set({ published: true })
  .where(eq(Post.authorId, 1))
  .returning();
```

## Delete

```ts
await db.delete(Post).where(eq(Post.id, 1));

// Delete all (be careful!)
await db.delete(Post);

// Delete with returning
const deleted = await db.delete(Post)
  .where(eq(Post.authorId, 1))
  .returning();
```

---

## Filtering

```ts
import { eq, ne, gt, gte, lt, lte, like, ilike, inArray, notInArray, between, isNull, isNotNull, and, or, not } from 'astro:db';

// Equality
await db.select().from(Post).where(eq(Post.id, 1));

// Like (pattern match)
await db.select().from(Post).where(like(Post.title, '%Hello%'));

// Multiple conditions
await db.select().from(Post).where(and(
  eq(Post.authorId, 1),
  like(Post.title, '%search%'),
  eq(Post.published, true),
));

// OR conditions
await db.select().from(Post).where(or(
  eq(Post.authorId, 1),
  eq(Post.authorId, 2),
));

// IN array
await db.select().from(Post).where(
  inArray(Post.id, [1, 2, 3])
);

// NULL checks
await db.select().from(Post).where(isNull(Post.metadata));
await db.select().from(Post).where(isNotNull(Post.metadata));

// Greater than / Less than
await db.select().from(Post).where(gt(Post.id, 10));
```

---

## Joins

```ts
// Inner join
const result = await db.select()
  .from(Post)
  .innerJoin(Author, eq(Post.authorId, Author.id));

// Result: Array<{ Post: {...}, Author: {...} }>

// Left join
await db.select()
  .from(Post)
  .leftJoin(Author, eq(Post.authorId, Author.id));

// Multiple joins
await db.select()
  .from(Comment)
  .innerJoin(Post, eq(Comment.postId, Post.id))
  .innerJoin(Author, eq(Comment.authorId, Author.id));
```

Available: `innerJoin()`, `leftJoin()`, `rightJoin()`, `fullJoin()`.

---

## Aggregation

```ts
import { count, countDistinct, avg, sum, max, min } from 'astro:db';

const [{ total }] = await db.select({
  total: count(),
}).from(Post);

const [{ uniqueAuthors }] = await db.select({
  uniqueAuthors: countDistinct(Post.authorId),
}).from(Post);
```

Group by:

```ts
const result = await db.select({
  authorId: Post.authorId,
  postCount: count(),
}).from(Post).groupBy(Post.authorId);
```

---

## Raw SQL

```ts
import { sql } from 'astro:db';

// Raw query
const result = await db.run(sql`
  SELECT * FROM Post WHERE title LIKE ${'%search%'}
`);

// With template
const searchTerm = 'Hello';
const posts = await db.run(sql`
  SELECT title, body FROM Post
  WHERE title LIKE ${'%' + searchTerm + '%'}
  ORDER BY title ASC
`);
```

---

## Batch Transactions

Combine multiple queries into a single network request with automatic rollback:

```ts
export default async function () {
  const queries = [];
  for (let i = 0; i < 100; i++) {
    queries.push(
      db.insert(Comment).values({
        postId: 1, authorId: 1, body: `Comment ${i}`,
      })
    );
  }
  await db.batch(queries); // Single network call
}
```

If any query fails, all are rolled back.

---

## Ordering & Limits

```ts
// Order by
await db.select().from(Post).orderBy(Post.title);
await db.select().from(Post).orderBy(Post.id, 'desc');

// Limit & offset
await db.select().from(Post).limit(10);
await db.select().from(Post).limit(10).offset(20);
```

---

## Full Drizzle Operator Reference

```ts
// Comparison
eq, ne, gt, gte, lt, lte, between

// Pattern matching
like, ilike, notLike, notIlike

// Arrays
inArray, notInArray

// NULL
isNull, isNotNull

// Logical
and, or, not

// Aggregation
count, countDistinct, avg, avgDistinct, sum, sumDistinct, max, min

// SQL
sql
```
