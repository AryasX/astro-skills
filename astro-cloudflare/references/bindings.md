# Cloudflare Bindings (KV, D1, R2, DO, Queues)

## Environment Variables

Define in `wrangler.jsonc`:

```jsonc
{
  "vars": {
    "API_URL": "https://api.example.com"
  }
}
```

Secrets via CLI (not in config file):
```bash
npx wrangler secret put DB_PASSWORD
```

Local secrets in `.dev.vars`:
```
DB_PASSWORD=myPassword
```

### Access from Code

```ts
// Primary way (v13+):
import { env } from 'cloudflare:workers';
const apiUrl = env.API_URL;

// Also compatible:
import { API_URL } from 'astro:env/server';
```

## KV (Key-Value Storage)

```jsonc
// wrangler.jsonc
{
  "kv_namespaces": [
    { "binding": "MY_KV", "id": "<namespace_id>" }
  ]
}
```

```ts
import { env } from 'cloudflare:workers';

// Get
const value = await env.MY_KV.get('key');

// Get with type (json, text, arrayBuffer, stream)
const data = await env.MY_KV.get('key', 'json');

// Put
await env.MY_KV.put('key', 'value');

// Put with options
await env.MY_KV.put('key', 'value', {
  expirationTtl: 3600,       // seconds
  expiration: Math.floor(Date.now() / 1000) + 3600, // UNIX timestamp in seconds
  metadata: { owner: 'user1' },
});

// List keys
const { keys } = await env.MY_KV.list({ prefix: 'user:', limit: 100 });

// Delete
await env.MY_KV.delete('key');

// Get with metadata
const { value, metadata } = await env.MY_KV.getWithMetadata('key', 'json');
```

Note: KV is eventually consistent (~60s global). Writes are immediate within same region.

## D1 (SQLite Database)

```jsonc
{
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<db_id>" }
  ]
}
```

```ts
import { env } from 'cloudflare:workers';

// Query
const { results } = await env.DB.prepare(
  'SELECT * FROM users WHERE id = ?'
).bind(userId).all();

// Single result
const user = await env.DB.prepare(
  'SELECT * FROM users WHERE id = ?'
).bind(userId).first();

// Insert/update/delete
const { success } = await env.DB.prepare(
  'INSERT INTO users (name, email) VALUES (?, ?)'
).bind(name, email).run();

// Batch
const stmts = [
  env.DB.prepare('INSERT INTO users VALUES (?)').bind('Alice'),
  env.DB.prepare('INSERT INTO users VALUES (?)').bind('Bob'),
];
await env.DB.batch(stmts);

// Raw results (no parsing)
const { results } = await env.DB.prepare('SELECT * FROM users').raw();
```

For Astro DB + D1 together, use Astro DB on the remote side with `--remote` flag.

## R2 (Object Storage)

```jsonc
{
  "r2_buckets": [
    { "binding": "MY_BUCKET", "bucket_name": "my-bucket" }
  ]
}
```

```ts
import { env } from 'cloudflare:workers';

// Put
await env.MY_BUCKET.put('path/to/file.txt', 'Hello world');

// Get
const object = await env.MY_BUCKET.get('path/to/file.txt');
const text = await object.text();
const buffer = await object.arrayBuffer();

// Delete
await env.MY_BUCKET.delete('path/to/file.txt');

// List
const { objects } = await env.MY_BUCKET.list({ prefix: 'uploads/', limit: 100 });

// Head (metadata only)
const { size, etag, customMetadata } = await env.MY_BUCKET.head('path/to/file.txt');
```

## Durable Objects

```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "MY_DO", "class_name": "MyDurableObject" }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["MyDurableObject"]  // SQLite-backed (recommended)
    }
  ]
}
```

```ts
import { env } from 'cloudflare:workers';

// RPC Pattern (Recommended 2024+) — direct method calls, full type safety:
const id = env.MY_DO.idFromName('instance-name');
const stub = env.MY_DO.get(id);
const result = await stub.someMethod();  // Direct RPC call, no serialization

// Legacy fetch pattern (pre-2024):
// const response = await stub.fetch('https://internal/endpoint');
```

For full Durable Object class definition, use a [custom entrypoint](../setup.md#custom-entrypoint-durable-objects-queues).

## Queues

```jsonc
{
  "queues": {
    "producers": [
      { "binding": "MY_QUEUE", "queue": "my-queue" }
    ],
    "consumers": [
      { "queue": "my-queue" }
    ]
  }
}
```

```ts
// Send
await env.MY_QUEUE.send({ type: 'email', to: 'user@example.com' });

// Send batch
await env.MY_QUEUE.sendBatch([
  { body: { type: 'email', to: 'a@example.com' } },
  { body: { type: 'email', to: 'b@example.com' } },
]);
```

Queue consumption requires a [custom entrypoint](../setup.md#custom-entrypoint-durable-objects-queues).

## The `cf` Object (Request Metadata)

```ts
// In component frontmatter (v13+):
const cf = Astro.request.cf;
const country = cf?.country;       // ISO country code
const region = cf?.region;         // Region code
const city = cf?.city;             // City name
const timezone = cf?.timezone;     // IANA timezone
const colo = cf?.colo;             // Cloudflare colo code
const tlsVersion = cf?.tlsVersion; // TLS version
```

## Execution Context

```ts
// In component frontmatter:
const ctx = Astro.locals.cfContext;
ctx.waitUntil(someAsyncTask());  // Extend Worker lifetime for background tasks
```

## TypeScript Types

Generate types from Wrangler config:

```bash
npx wrangler types
```

This generates `.wrangler/types/runtime.d.ts` with typed bindings.

Configure `tsconfig.json` to include the generated types:

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "types": ["@cloudflare/workers-types"]
  },
  "include": [".wrangler/types/**/*.ts"]
}
```

Then import and use the typed `Env` interface in your code:

```ts
import type { Env } from './.wrangler/types/runtime';

// Use in custom entrypoint:
export default {
  fetch(request: Request, env: Env, ctx: ExecutionContext) {
    // env is now fully typed with your bindings
  },
};
```

Automate type generation in `package.json`:

```json
{
  "scripts": {
    "dev": "wrangler types && astro dev",
    "build": "wrangler types && astro check && astro build",
    "preview": "wrangler types && astro preview"
  }
}
```

## Astro DB on Cloudflare

```bash
astro build --remote
```

Or configure in `package.json`:

```json
{
  "scripts": {
    "build": "astro build --remote"
  }
}
```

Set `ASTRO_DB_REMOTE_URL` and `ASTRO_DB_APP_TOKEN` for Turso connection.
The `--remote` flag is needed for both local dev and production builds.

---

## Beyond this Skill: Deep-Dive Topics in the Cloudflare Skill

This skill covers Astro-specific Cloudflare integration. For deeper coverage, load the **cloudflare** skill, which includes:

| Topic | Cloudflare Skill Reference |
|-------|---------------------------|
| D1 Sessions API (read-after-write consistency) | `cloudflare/references/d1/api.md` |
| D1 Read Replicas | `cloudflare/references/d1/api.md` |
| D1 Error Handling | `cloudflare/references/d1/api.md` |
| D1 SQL Migrations | `cloudflare/references/d1/configuration.md` |
| DO Storage SQL API (`ctx.storage.sql`) | `cloudflare/references/do-storage/README.md` |
| DO Alarms (`setAlarm`, `alarm()`) | `cloudflare/references/durable-objects/api.md` |
| DO WebSocket Hibernation | `cloudflare/references/durable-objects/api.md` |
| DO PITR (Point-in-Time Recovery) | `cloudflare/references/do-storage/README.md` |
| Cache API (`caches.default`) | `cloudflare/references/workers/api.md` |
| HTMLRewriter | `cloudflare/references/workers/api.md` |
| Workers Limits (CPU, subrequests) | `cloudflare/references/workers/gotchas.md` |
| Drizzle ORM with D1 | `cloudflare/references/d1/configuration.md` |
