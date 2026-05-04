# Querying & Rendering Content

## Querying Collections

```ts
import { getCollection, getEntry, getEntries } from 'astro:content';
```

### Get all entries

```ts
const posts = await getCollection('blog');
```

**Note:** Sort order is non-deterministic. Always sort yourself:

```ts
const posts = (await getCollection('blog'))
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
```

### Get single entry

```ts
const post = await getEntry('blog', 'my-post-slug');
if (!post) {
  return Astro.redirect('/404');
}
```

### Get multiple entries (for references)

```ts
const related = await getEntries(post.data.relatedPosts);
```

### Filtering

```ts
// Filter by data properties
const published = await getCollection('blog', ({ data }) => !data.draft);

// Filter by ID prefix (sub-directory)
const englishDocs = await getCollection('docs', ({ id }) => id.startsWith('en/'));

// Conditional (dev vs prod)
const entries = await getCollection('blog', ({ data }) => {
  return import.meta.env.PROD ? !data.draft : true;
});
```

## Rendering Body Content

```astro
---
import { getEntry, render } from 'astro:content';

const entry = await getEntry('blog', 'post-1');
const { Content, headings } = await render(entry);
---

<h1>{entry.data.title}</h1>
<p>Published: {entry.data.pubDate.toDateString()}</p>

<!-- headings: Array<{ depth, slug, text }> — useful for ToC -->
<Content />
```

### MDX with Custom Components

```astro
---
import CustomCode from '../components/CustomCode.astro';
const { Content } = await render(entry);
---
<Content components={{ code: CustomCode }} />
```

## Generating Routes from Content

### Static (build-time)

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---
```

**Do NOT use `Astro.site` inside `getStaticPaths()`** — deprecated in v6. Use `import.meta.env.SITE`.

### SSR (on-demand)

For large collections where you don't want to prerender all pages:

```astro
---
export const prerender = false;

import { getEntry } from 'astro:content';

const post = await getEntry('blog', Astro.params.slug);
if (!post) return Astro.redirect('/404');

const { Content } = await render(post);
---
```

## Type-Safe Props

```ts
import type { CollectionEntry } from 'astro:content';

interface Props {
  post: CollectionEntry<'blog'>;  // Fully typed based on schema
}

const { post } = Astro.props;
// post.data.title — typed as string
// post.data.pubDate — typed as Date
```

## CollectionEntry Properties

```ts
interface CollectionEntry<T> {
  id: string;                     // Unique slug/ID
  collection: string;             // Collection name
  data: InferEntrySchema<T>;      // Typed from Zod schema
  body?: string;                  // Raw body (Markdown, MDX, etc.)
  rendered?: {                    // From render()
    html: string;
    metadata?: { headings: Array<{ depth: number; slug: string; text: string }> };
  };
  digest?: string;                // Content digest for caching
}
```

## Accessing Referenced Data

```ts
// Schema has: author: reference('authors')
const post = await getEntry('blog', 'my-post');

// Resolve single reference
const author = await getEntry(post.data.author);
console.log(author.data.name);

// Resolve array of references: relatedPosts: z.array(reference('blog'))
const related = await getEntries(post.data.relatedPosts);
for (const p of related) {
  console.log(p.data.title);
}
```
