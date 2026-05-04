# View Transitions

Client-side SPA navigation with smooth page transitions.

## Enable

```astro
---
// In a layout or page
import { ClientRouter } from 'astro:transitions';
---
<html>
  <head>
    <ClientRouter />
    <!-- Optional: custom transition animations -->
    <style>
      ::view-transition-old(root) { animation: fade-out 0.3s; }
      ::view-transition-new(root) { animation: fade-in 0.3s; }
    </style>
  </head>
  <body>
    <slot />
  </body>
</html>
```

**Important:** `<ViewTransitions />` was **removed in v6**. Use `<ClientRouter />`.

## Programmatic Navigation

```astro
---
import { navigate } from 'astro:transitions/client';
---
<button onclick={() => navigate('/next-page')}>
  Go to next page
</button>
```

```ts
// In framework component or script
import { navigate } from 'astro:transitions/client';

await navigate('/dashboard');

// With options
navigate('/dashboard', {
  history: 'replace',  // 'auto' | 'push' | 'replace'
  formData: new FormData(form),
});
```

## Transition Events

Listen for transition lifecycle events:

```ts
document.addEventListener('astro:before-preparation', () => {
  // Start loading indicator
});

document.addEventListener('astro:after-preparation', () => {
  // New page is ready, about to swap
});

document.addEventListener('astro:before-swap', () => {
  // About to swap old → new DOM
  // Can mutate newDocument before swap
});

document.addEventListener('astro:after-swap', () => {
  // New page is visible
  // Re-initialize components, analytics
});

document.addEventListener('astro:page-load', () => {
  // Page loaded (first load + subsequent navigations)
});
```

**Note:** Event name constants like `TRANSITION_BEFORE_SWAP` are **removed in v6**. Use string literals.

## Persist Elements Across Navigation

```html
<nav transition:persist>
  <!-- This element persists across page navigations -->
</nav>
```

Useful for:
- Persistent audio players
- Navigation bars that shouldn't re-render
- Video embeds that should keep playing

## Transition Directives

```
transition:name="unique-name"    — Tag element for morph animation
transition:animate="fade"        — Use built-in animation
transition:animate="slide"       — Slide from direction
transition:animate="none"        — Disable animation for this element
transition:persist               — Keep element across navigations
```

## Handling Forms with View Transitions

```astro
<form method="POST" action="/search" transition:persist>
  <input type="search" name="q" />
  <button type="submit">Search</button>
</form>
```

Form submissions are intercepted by ClientRouter and handled via fetch, enabling smooth transitions.

## Fallback for No JavaScript

View Transitions degrade gracefully. Without JavaScript, pages load normally with full browser navigation.
