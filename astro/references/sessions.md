# Sessions

Server-side state between requests. Added in astro@5.7, compatible with v6.

## Quick Usage

```astro
---
// In page or component
export const prerender = false; // Must be SSR

// Get
const cart = await Astro.session?.get('cart');

// Set
await Astro.session?.set('cart', ['item-1', 'item-2']);

// Regenerate session ID (e.g., after login)
await Astro.session?.regenerate();

// Destroy session
await Astro.session?.destroy();
---
```

## In Endpoints / Actions

```ts
// Endpoint
export async function POST(context: APIContext) {
  const cart = await context.session?.get('cart') || [];
  cart.push(newItem);
  await context.session?.set('cart', cart);
  return Response.json(cart);
}

// Action
export const server = {
  addToCart: defineAction({
    input: z.object({ item: z.string() }),
    handler: async ({ item }, context) => {
      const cart = await context.session?.get('cart') || [];
      cart.push(item);
      await context.session?.set('cart', cart);
      return cart;
    },
  }),
};
```

## In Middleware

```ts
export const onRequest = defineMiddleware(async (context, next) => {
  await context.session?.set('lastVisit', new Date());
  return next();
});
```

Note: Sessions not supported in **edge middleware**.

---

## Configuration

```js
// astro.config.mjs
import { defineConfig, sessionDrivers } from 'astro/config';

export default defineConfig({
  session: {
    driver: sessionDrivers.redis({ url: process.env.REDIS_URL }),
    cookie: {
      name: 'sid',
      secure: true,
      httpOnly: true,
      sameSite: 'lax',
      path: '/',
      maxAge: 3600,
    },
    ttl: 3600,  // Seconds until idle session expires
  },
});
```

### Auto-configured drivers

| Adapter | Driver | Notes |
|---------|--------|-------|
| `@astrojs/node` | Filesystem | `.astro/sessions/` |
| `@astrojs/cloudflare` | KV | Auto-provisioned, binding: `SESSION` |
| `@astrojs/netlify` | Netlify Blobs | Auto-configured |
| `@astrojs/vercel` | Must configure manually | `sessionDrivers.redis()` or custom |

### Available drivers

```js
sessionDrivers.redis({ url: '...' })
sessionDrivers.netlifyBlobs({ name: 'sessions' })
sessionDrivers.cloudflareKV({ binding: 'MY_KV' })
// Custom: { entrypoint: './src/session-driver.ts' }
```

---

## Typed Sessions

```ts
// src/env.d.ts
declare namespace App {
  interface SessionData {
    user: {
      id: string;
      name: string;
      email: string;
    };
    cart: string[];
    lastVisit: Date;
  }
}
```

Now `session.get('cart')` returns `string[] | undefined` with full type checking.

---

## Serialization

Session data is serialized using **devalue** (same library as actions and content collections). Supported types:

- Strings, numbers, booleans
- Date, Map, Set, URL
- Arrays, plain objects
- BigInt, RegExp, Infinity, NaN

Unsupported: Functions, class instances, Symbols.
