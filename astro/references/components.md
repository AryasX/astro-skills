# Components

## .astro Component Syntax

```astro
---
// Frontmatter: runs at build time or on server
import Layout from '../layouts/Base.astro';

interface Props {
  title: string;
  items: string[];
}

const { title, items } = Astro.props;
---

<Layout title={title}>
  <h1 class="heading">{title}</h1>

  <ul>
    {items.map(item => <li>{item}</li>)}
  </ul>

  <slot />  <!-- Children content -->
  <slot name="sidebar" />  <!-- Named slot -->
</Layout>

<style>
  /* Scoped by default — only affects this component */
  .heading { color: var(--primary); }
</style>
```

## Props

```astro
---
// Typed props
interface Props {
  title: string;
  count?: number;      // Optional
  callback: () => void;  // Functions (client islands only)
  children?: any;       // Slot content type
}

const { title, count = 0 } = Astro.props;
---
```

## Layouts

```astro
---
// src/layouts/Base.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
---
<!doctype html>
<html>
  <head>
    <title>{title}</title>
  </head>
  <body>
    <slot />  <!-- Page content goes here -->
  </body>
</html>
```

Usage in pages:

```astro
---
import Layout from '../layouts/Base.astro';
---
<Layout title="Home">
  <h1>Welcome</h1>
</Layout>
```

## Template Directives

```astro
{value}               — Render value (strings auto-escaped)
<div set:html={html} /> — Set inner HTML (unescaped)
<div set:text={text} /> — Set text content (escaped)
```

## Conditional Rendering

```astro
{condition ? <Show /> : <Hide />}
{condition && <Show />}
{items.map(item => <li>{item}</li>)}
```

## Component File Types

| Directory | Convention |
|-----------|-----------|
| `src/components/` | Reusable components (.astro, .tsx, .vue, etc.) |
| `src/layouts/` | Layout wrappers (optional convention) |
| `src/pages/` | Routes (required) |
