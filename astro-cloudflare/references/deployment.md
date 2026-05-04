# Cloudflare Deployment

## Build & Deploy

```bash
# Build
astro build

# Deploy
npx wrangler deploy
```

## Local Preview (Production Test)

```bash
astro build && astro preview
```

Preview runs on `workerd` — same runtime as production. For clearer error messages, disable minification:

```js
export default defineConfig({
  adapter: cloudflare(),
  vite: { build: { minify: false } },
});
```

## Environment-Specific Deployment

Build once per environment:

```bash
CLOUDFLARE_ENV=staging astro build && wrangler deploy
CLOUDFLARE_ENV=production astro build && wrangler deploy
```

The `CLOUDFLARE_ENV` variable determines which Wrangler environment configuration is used during the build.

## CI/CD (GitHub Actions)

```yaml
name: Deploy to Cloudflare Workers

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npm run build
      - run: npx wrangler deploy
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

For environment-specific deploys:

```yaml
- run: CLOUDFLARE_ENV=production npm run build
- run: npx wrangler deploy
```

## Bindings on Multiple Environments

```jsonc
// wrangler.jsonc
{
  "name": "my-astro-app",
  "env": {
    "staging": {
      "kv_namespaces": [
        { "binding": "MY_KV", "id": "<staging-id>" }
      ]
    },
    "production": {
      "kv_namespaces": [
        { "binding": "MY_KV", "id": "<prod-id>" }
      ]
    }
  }
}
```

## Remote Bindings in Development

```js
adapter: cloudflare({ remoteBindings: true }),
```

```bash
CLOUDFLARE_ENV=staging astro dev
```

Access production/remote resources during local development.

## Build Output

The build outputs to `dist/`. On deploy, Wrangler uploads:
- `dist/` — static assets (served via Workers static assets)
- Worker code — bundled SSR handler

Static assets are served directly by Cloudflare's CDN where possible, falling back to the Worker for on-demand rendered routes.

## Multi-Worker / Auxiliary Workers

```js
adapter: cloudflare({
  auxiliaryWorkers: [
    { config: './worker-a/wrangler.jsonc' },
    { config: './worker-b/wrangler.jsonc' },
  ],
}),
```

Run additional Workers alongside your main Astro Worker during development.
