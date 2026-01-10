# Convex Actions Reference

Actions run arbitrary code, including calling external APIs. They cannot access the database directly.

## Basic Action

```typescript
import { action } from "./_generated/server";
import { v } from "convex/values";

export const fetchWeather = action({
  args: { city: v.string() },
  returns: v.object({
    temperature: v.number(),
    conditions: v.string(),
  }),
  handler: async (ctx, args) => {
    const response = await fetch(
      `https://api.weather.com/v1/${args.city}`
    );
    const data = await response.json();
    return {
      temperature: data.temp,
      conditions: data.conditions,
    };
  },
});
```

## Using Node.js Modules

Add `"use node";` at the top of files using Node.js built-in modules:

**Important:** Files with `"use node";` should NEVER contain mutations or queries — only actions. Node actions can only be called from the client or from other actions.

```typescript
"use node";

import { action } from "./_generated/server";
import { v } from "convex/values";
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export const generateText = action({
  args: { prompt: v.string() },
  returns: v.string(),
  handler: async (ctx, args) => {
    const response = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: [{ role: "user", content: args.prompt }],
    });
    return response.choices[0].message.content ?? "";
  },
});
```

## Internal Action

```typescript
import { internalAction } from "./_generated/server";

export const processWebhook = internalAction({
  args: { payload: v.any() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Only callable from other Convex functions
    return null;
  },
});
```

## Actions Cannot Access ctx.db

**Wrong:**
```typescript
export const badAction = action({
  handler: async (ctx, args) => {
    await ctx.db.insert("users", { name: "test" });  // ERROR!
  },
});
```

**Correct:** Call mutations instead:
```typescript
import { internal } from "./_generated/api";

export const goodAction = action({
  args: { name: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Call a mutation to write to database
    await ctx.runMutation(internal.users.create, { name: args.name });
    return null;
  },
});
```

## Calling Functions from Actions

```typescript
import { api, internal } from "./_generated/api";

export const processData = action({
  args: { userId: v.id("users") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Read data via query
    const user = await ctx.runQuery(api.users.get, { userId: args.userId });

    // Call external API
    const enrichedData = await fetch(`https://api.example.com/enrich/${user.email}`);

    // Write data via mutation
    await ctx.runMutation(internal.users.updateEnriched, {
      userId: args.userId,
      data: await enrichedData.json(),
    });

    return null;
  },
});
```

## Don't Call Actions from Actions (Usually)

**Only call an action from another action if crossing runtimes (V8 → Node).**

Otherwise, extract shared logic into a helper function:

```typescript
// Helper function (not a Convex function)
async function callExternalApi(data: string) {
  const response = await fetch("https://api.example.com", {
    method: "POST",
    body: data,
  });
  return response.json();
}

export const action1 = action({
  args: { data: v.string() },
  returns: v.any(),
  handler: async (ctx, args) => {
    return await callExternalApi(args.data);
  },
});

export const action2 = action({
  args: { data: v.string() },
  returns: v.any(),
  handler: async (ctx, args) => {
    return await callExternalApi(args.data);
  },
});
```

## HTTP Endpoints

Define in `convex/http.ts`:

```typescript
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";

const http = httpRouter();

http.route({
  path: "/webhook",
  method: "POST",
  handler: httpAction(async (ctx, req) => {
    const body = await req.json();

    // Process webhook...

    return new Response(JSON.stringify({ success: true }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  }),
});

http.route({
  path: "/health",
  method: "GET",
  handler: httpAction(async (ctx, req) => {
    return new Response("OK", { status: 200 });
  }),
});

export default http;
```

**Path registration:** Endpoints are registered at the exact path specified.
- `/api/webhook` → available at `/api/webhook`

### HTTP Action Request Methods

```typescript
handler: httpAction(async (ctx, req) => {
  const body = await req.json();       // Parse JSON body
  const text = await req.text();       // Get raw text
  const bytes = await req.bytes();     // Get raw bytes
  const url = new URL(req.url);        // Access URL/query params
  const headers = req.headers;         // Access headers

  return new Response(body, { status: 200 });
})
```

### Calling Functions from HTTP Actions

```typescript
http.route({
  path: "/api/users",
  method: "POST",
  handler: httpAction(async (ctx, req) => {
    const { name, email } = await req.json();

    const userId = await ctx.runMutation(api.users.create, { name, email });

    return new Response(JSON.stringify({ userId }), {
      status: 201,
      headers: { "Content-Type": "application/json" },
    });
  }),
});
```

## Environment Variables

Access via `process.env`:

```typescript
const apiKey = process.env.OPENAI_API_KEY;
const webhookSecret = process.env.WEBHOOK_SECRET;
```

Set in Convex dashboard under "Settings" → "Environment Variables".

## Limits

- Actions execute for max 10 minutes
- Function arguments/returns: max 8 MiB
- HTTP streaming output: max 20 MiB
