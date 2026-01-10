# Convex Project Setup Reference

How to initialize a new Convex project or add Convex to an existing project.

## Quick Start (New Project)

Create a new project with Convex pre-configured:

```bash
npm create convex@latest
```

This scaffolds a new project with Convex already set up.

## Adding Convex to Existing Projects

### Step 1: Install Convex

```bash
npm install convex
```

### Step 2: Initialize Convex

```bash
npx convex dev
```

This command:
- Prompts you to log in with GitHub
- Creates a new Convex project
- Saves deployment URLs to `.env.local`
- Creates the `convex/` folder for backend functions
- Continues running to sync functions with your dev deployment

Keep this running in a terminal while developing.

---

## Next.js Setup (App Router)

### Environment Variable

```
NEXT_PUBLIC_CONVEX_URL=<your-deployment-url>
```

Automatically created by `npx convex dev`.

### Provider Component

Create `app/ConvexClientProvider.tsx`:

```tsx
"use client";

import { ConvexProvider, ConvexReactClient } from "convex/react";
import { ReactNode } from "react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function ConvexClientProvider({ children }: { children: ReactNode }) {
  return <ConvexProvider client={convex}>{children}</ConvexProvider>;
}
```

### Layout Integration

In `app/layout.tsx`:

```tsx
import { ConvexClientProvider } from "./ConvexClientProvider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <ConvexClientProvider>{children}</ConvexClientProvider>
      </body>
    </html>
  );
}
```

---

## React + Vite Setup

### Environment Variable

```
VITE_CONVEX_URL=<your-deployment-url>
```

Automatically created by `npx convex dev`.

### Provider Setup

In `src/main.tsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";
import { ConvexProvider, ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(import.meta.env.VITE_CONVEX_URL as string);

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ConvexProvider client={convex}>
      <App />
    </ConvexProvider>
  </React.StrictMode>,
);
```

---

## Node.js / Scripts Setup

For server-side code, migrations, scripts, or serverless functions.

### Environment Variable

```
CONVEX_URL=<your-deployment-url>
```

### ConvexHttpClient

```typescript
import { ConvexHttpClient } from "convex/browser";
import { api } from "./convex/_generated/api";

const client = new ConvexHttpClient(process.env.CONVEX_URL!);

// Run a query
const data = await client.query(api.myModule.myQuery, { arg: "value" });

// Execute a mutation
await client.mutation(api.myModule.myMutation, { arg: "value" });
```

**Note:** `ConvexHttpClient` runs queries at a single point in time (non-reactive).
For reactive subscriptions in long-running servers, use `ConvexClient` instead.

---

## Project Structure After Setup

```
my-app/
├── convex/
│   ├── _generated/      # Auto-generated (don't edit)
│   │   ├── api.d.ts
│   │   ├── api.js
│   │   ├── dataModel.d.ts
│   │   └── server.d.ts
│   ├── schema.ts        # Database schema
│   └── myFunctions.ts   # Your backend functions
├── .env.local           # Convex URLs (gitignored)
└── ...
```

---

## Common Commands

- **`npx convex dev`** — Start dev server, sync functions
- **`npx convex deploy`** — Deploy to production
- **`npx convex codegen`** — Regenerate types without syncing
- **`npx convex dashboard`** — Open Convex dashboard

---

## ESLint Setup (Recommended)

Add Convex-specific linting rules. See `references/eslint.md` for full details.

```bash
npm i -D @convex-dev/eslint-plugin
```

```javascript
// eslint.config.js
import { defineConfig } from "eslint/config";
import convexPlugin from "@convex-dev/eslint-plugin";

export default defineConfig([
  ...convexPlugin.configs.recommended,
]);
```

Add scripts to `package.json`:

```json
{
  "scripts": {
    "lint": "eslint convex/",
    "lint:fix": "eslint convex/ --fix"
  }
}
```

---

## Environment Variables Summary

| Framework | Variable Name | Access Pattern |
|-----------|--------------|----------------|
| Next.js | `NEXT_PUBLIC_CONVEX_URL` | `process.env.NEXT_PUBLIC_CONVEX_URL` |
| Vite | `VITE_CONVEX_URL` | `import.meta.env.VITE_CONVEX_URL` |
| Node.js | `CONVEX_URL` | `process.env.CONVEX_URL` |

---

## First Backend Function

After setup, create your first function in `convex/`:

```typescript
// convex/tasks.ts
import { query, mutation } from "./_generated/server";
import { v } from "convex/values";

export const list = query({
  args: {},
  returns: v.array(v.object({
    _id: v.id("tasks"),
    _creationTime: v.number(),
    text: v.string(),
    completed: v.boolean(),
  })),
  handler: async (ctx) => {
    return await ctx.db.query("tasks").collect();
  },
});

export const create = mutation({
  args: { text: v.string() },
  returns: v.id("tasks"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("tasks", {
      text: args.text,
      completed: false,
    });
  },
});
```

---

## Using Functions in React

```tsx
import { useQuery, useMutation } from "convex/react";
import { api } from "../convex/_generated/api";

function TaskList() {
  const tasks = useQuery(api.tasks.list);
  const createTask = useMutation(api.tasks.create);

  if (tasks === undefined) return <div>Loading...</div>;

  return (
    <div>
      <button onClick={() => createTask({ text: "New task" })}>
        Add Task
      </button>
      {tasks.map((task) => (
        <div key={task._id}>{task.text}</div>
      ))}
    </div>
  );
}
```
