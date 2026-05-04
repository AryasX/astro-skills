# Sessions on Cloudflare

Cloudflare adapter automatically configures Workers KV for session storage.

## Auto-Provisioned (Default)

No setup needed. When you deploy, Wrangler auto-provisions a KV namespace named `SESSION`.

```astro
---
export const prerender = false;
const cart = await Astro.session?.get('cart');
await Astro.session?.set('cart', ['item-1']);
---
```

## Custom Binding Name

```js
// astro.config.mjs
adapter: cloudflare({
  sessionKVBindingName: 'MY_SESSION',
}),
```

```jsonc
// wrangler.jsonc (optional)
{
  "kv_namespaces": [
    { "binding": "MY_SESSION" }
  ]
}
```

## Session Configuration

```js
// astro.config.mjs
export default defineConfig({
  adapter: cloudflare({
    sessionKVBindingName: 'SESSION',
  }),
  session: {
    cookie: {
      name: 'sid',
      secure: true,
      httpOnly: true,
      sameSite: 'lax',
      path: '/',
      maxAge: 86400,  // 24 hours
    },
    ttl: 86400,       // Idle session timeout (seconds)
  },
});
```

## KV Consistency Note

Cloudflare KV is **eventually consistent** — writes are available immediately within the same region but may take up to 60 seconds to propagate globally. This is typically not an issue since users are unlikely to switch regions between requests, but consider for scenarios like VPN users or global load-balanced applications.

## Manual KV Control

If you need more control over the session KV namespace:

```bash
# Create namespace
npx wrangler kv:namespace create SESSION

# Copy the ID into wrangler.jsonc
```

```jsonc
{
  "kv_namespaces": [
    { "binding": "SESSION", "id": "abc123..." }
  ]
}
```

Then set in adapter:

```js
adapter: cloudflare({
  sessionKVBindingName: 'SESSION',
}),
```
