# Islands Architecture & Client Directives

Astro renders everything as static HTML by default. Client directives control when JavaScript hydrates on the client.

## Client Directives

```
<MyReactComponent client:load />
    → Hydrate immediately on page load

<MyVueComponent client:idle />
    → Hydrate when browser is idle (requestIdleCallback)

<MySvelteComponent client:visible />
    → Hydrate when element enters viewport (IntersectionObserver)

<MyComponent client:media="(max-width: 768px)" />
    → Hydrate when media query matches

<MyComponent client:only="react" />
    → Skip server render entirely, client-side only
    → Must specify framework: react, vue, svelte, solid, preact

<MyComponent client:only />
    → Client only, auto-detects framework from import
```

## Framework Integration

```bash
npx astro add react
npx astro add vue
npx astro add svelte
npx astro add solid
npx astro add preact
npx astro add alpinejs      # No client directive needed
```

## Using Framework Components

```astro
---
import ReactCounter from '../components/Counter.tsx';
import VueTodos from '../components/Todos.vue';
---
<ReactCounter client:load />
<VueTodos client:visible />
```

## Sharing State Between Islands

Use framework-agnostic patterns:

**Custom events:**
```ts
document.addEventListener('cart-updated', (e) => { ... });
window.dispatchEvent(new CustomEvent('cart-updated', { detail: data }));
```

**Nano Stores (recommended):**
```bash
npm install nanostores @nanostores/react
```

```ts
// src/store/cart.ts
import { atom } from 'nanostores';
export const cartItems = atom<string[]>([]);
```

```tsx
// In React component
import { useStore } from '@nanostores/react';
import { cartItems } from '../store/cart';

export function CartButton() {
  const items = useStore(cartItems);
  return <button>Cart ({items.length})</button>;
}
```

## Passing Props to Framework Components

Props are serialized. Plain objects, arrays, strings, numbers, booleans are fine. **No functions, classes, or complex objects** — they can't be serialized from server to client.

```astro
---
// ✅ OK
const items = ['a', 'b', 'c'];
// ❌ Cannot pass: callbacks, class instances, Maps, Sets
---
<MyComponent client:load items={items} />
```

If you need to pass a function, define it in a `<script>` tag that hydrates with the component.

## Server Islands

Defer rendering of a component to the server. Just add `server:defer` to any component:

```astro
---
// Avatar.astro
const userSession = Astro.cookies.get('session');
const avatarURL = await getUserAvatar(userSession);
---
<img alt="User avatar" src={avatarURL} />
```

```astro
---
import Avatar from '../components/Avatar.astro';
---
<Avatar server:defer />
```

Fallback content shown while the island loads:

```astro
<Avatar server:defer>
  <div slot="fallback">Loading avatar...</div>
</Avatar>
```

## Alpine.js (no client directive needed)

```astro
---
// No directive, Alpine auto-initializes
---
<div x-data="{ count: 0 }">
  <button @click="count++">Increment</button>
  <span x-text="count"></span>
</div>
```
