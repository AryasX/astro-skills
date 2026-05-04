# Astro Actions

Type-safe RPC for client-server communication. Define once, call from anywhere.

## Define actions

```ts
// src/actions/index.ts
import { defineAction, ActionError } from 'astro:actions';
import { z } from 'astro/zod';

export const server = {
  subscribe: defineAction({
    accept: 'form',          // 'form' or 'json' (default)
    input: z.object({
      email: z.email(),       // Zod 4: z.email() directly
      name: z.string().min(2, { error: 'Name must be at least 2 characters' }),
    }),
    handler: async ({ email, name }, context) => {
      // Server-side logic
      // Access: context.cookies, context.locals, context.session
      await saveToDatabase({ email, name });
      return { success: true };
    },
  }),

  likePost: defineAction({
    input: z.object({ postId: z.string() }),
    handler: async (input) => {
      if (!input.postId) {
        throw new ActionError({ code: 'BAD_REQUEST' });
      }
      const count = await incrementLikes(input.postId);
      return { likes: count };
    },
  }),
};
```

## Organizing actions

```ts
// src/actions/user.ts
import { defineAction } from 'astro:actions';

export const user = {
  getUser: defineAction({ /* ... */ }),
  createUser: defineAction({ /* ... */ }),
};
```

```ts
// src/actions/index.ts
import { user } from './user';

export const server = {
  subscribe: defineAction({ /* ... */ }),
  user,  // Nested: actions.user.getUser()
};
```

## Call from client

```ts
import { actions, isInputError } from 'astro:actions';

// Standard call
const { data, error } = await actions.subscribe({ email: '...', name: '...' });
if (error) {
  console.error(error);
  return;
}
console.log(data);  // { success: true }

// With FormData
const formData = new FormData(form);
const { data, error } = await actions.subscribe(formData);

// Skip error handling (throws on error)
const data = await actions.subscribe.orThrow({ email: '...', name: '...' });
```

## Handle input errors

```ts
import { actions, isInputError } from 'astro:actions';

const { error } = await actions.subscribe(formData);
if (isInputError(error)) {
  // error.fields.email = string[] of error messages
  if (error.fields.email) {
    showError('email', error.fields.email.join(', '));
  }
}
```

## HTML form action (zero JS)

```astro
---
import { actions, isInputError } from 'astro:actions';

const result = Astro.getActionResult(actions.subscribe);
const inputErrors = isInputError(result?.error) ? result.error.fields : {};
---

{result?.data && <p class="success">Subscribed!</p>}
{result?.error && !isInputError(result?.error) && <p class="error">Server error</p>}

<form method="POST" action={actions.subscribe}>
  <label>
    Email
    <input type="email" name="email" required />
  </label>
  {inputErrors.email && <p class="error">{inputErrors.email}</p>}

  <label>
    Name
    <input type="text" name="name" required />
  </label>
  {inputErrors.name && <p class="error">{inputErrors.name}</p>}

  <button type="submit">Subscribe</button>
</form>
```

## Redirect on success

```astro
---
import { actions } from 'astro:actions';

const result = Astro.getActionResult(actions.createProduct);
if (result?.data) {
  return Astro.redirect(`/products/${result.data.id}`);
}
---
```

## Form validators

When `accept: 'form'`:

| Input type | Validator |
|-----------|-----------|
| `<input type="text">` | `z.string()` |
| `<input type="number">` | `z.number()` |
| `<input type="checkbox">` | `z.coerce.boolean()` |
| `<input type="file">` | `z.instanceof(File)` |
| Multiple inputs same name | `z.array(z.string())` |

Empty inputs become `null` (except arrays and booleans).

## Discriminated union (conditional validation)

```ts
input: z.discriminatedUnion('type', [
  z.object({
    type: z.literal('create'),
    name: z.string(),
  }),
  z.object({
    type: z.literal('update'),
    id: z.number(),
    name: z.string(),
  }),
]),
```

## Security

### In handler

```ts
handler: async (input, context) => {
  if (!context.locals.user) {
    throw new ActionError({ code: 'UNAUTHORIZED' });
  }
}
```

### In middleware

```ts
import { getActionContext } from 'astro:actions';

export const onRequest = defineMiddleware(async (context, next) => {
  const { action } = getActionContext(context);
  if (action?.calledFrom === 'rpc') {
    if (!context.cookies.has('user-session')) {
      return new Response('Forbidden', { status: 403 });
    }
  }
  return next();
});
```

## Server-to-server calls

```astro
---
// In Astro component or endpoint
const result = await Astro.callAction(actions.likePost, { postId: '123' });
const result = await context.callAction(actions.likePost, { postId: '123' });
---
```

## ActionError codes

```ts
throw new ActionError({ code: 'NOT_FOUND', message: 'Post not found' });
throw new ActionError({ code: 'UNAUTHORIZED' });
throw new ActionError({ code: 'BAD_REQUEST' });
throw new ActionError({ code: 'INTERNAL_SERVER_ERROR' });
```

## Persist action results across redirects (POST/Redirect/GET)

Use middleware + sessions to persist results:

```ts
// src/middleware.ts
import { getActionContext } from 'astro:actions';

export const onRequest = defineMiddleware(async (context, next) => {
  const { action, setActionResult, serializeActionResult } = getActionContext(context);
  if (action?.calledFrom === 'form') {
    const result = await action.handler();
    context.session?.set('actionResult', serializeActionResult(result));
    return context.redirect(context.originPathname);
  }
  const saved = await context.session?.get('actionResult');
  if (saved) {
    setActionResult(action.name, saved);
    context.session?.delete('actionResult');
  }
  return next();
});
```
