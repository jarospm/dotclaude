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

## When to Use This Structure

**Single-repo (React at root + `/convex`)** is recommended for:
- Single web application
- Solo developers or small teams
- Getting started quickly

This is the official Convex quickstart pattern and works well for most projects.

For multi-platform apps (web + mobile), consider the Turborepo monorepo template:
https://www.convex.dev/templates/monorepo

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
│   ├── tsconfig.json    # Convex-specific TS config (isolated)
│   ├── schema.ts        # Database schema
│   └── myFunctions.ts   # Your backend functions
├── src/                 # Frontend code
├── .env.local           # Convex URLs (gitignored)
├── .prettierignore      # Ignore generated files
├── tsconfig.json        # Frontend TS config
└── eslint.config.js     # Separate rules for src/ and convex/
```

---

## TypeScript Configuration

Convex generates `convex/tsconfig.json` — a **separate** TypeScript project.
This is intentional: Convex code runs in a different runtime than your frontend.

### Update Scripts to Cover Both

The default `typecheck` script only checks frontend code. Update `package.json`:

```json
{
  "scripts": {
    "typecheck": "tsc -b && tsc -p convex",
    "build": "tsc -b && tsc -p convex && vite build"
  }
}
```

### Why Not Project References?

Convex's tsconfig requires `noEmit: true`, which is incompatible with TypeScript
project references (`composite: true`). Run both checks sequentially instead.

---

## Common Commands

- **`npx convex dev`** — Start dev server, sync functions
- **`npx convex deploy`** — Deploy to production
- **`npx convex codegen`** — Regenerate types without syncing
- **`npx convex dashboard`** — Open Convex dashboard

---

## ESLint Setup (React + Vite + Convex)

Frontend and backend run in **different environments** — they need separate configs.
See `references/eslint.md` for full details on Convex-specific rules.

### Install

```bash
npm i -D @convex-dev/eslint-plugin
```

### Complete eslint.config.js

```javascript
import js from '@eslint/js';
import globals from 'globals';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import tseslint from 'typescript-eslint';
import convexPlugin from '@convex-dev/eslint-plugin';
import { defineConfig, globalIgnores } from 'eslint/config';

export default defineConfig([
  globalIgnores(['dist', 'convex/_generated']),

  // Frontend: Browser environment with React
  {
    files: ['src/**/*.{ts,tsx}'],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommended,
      reactHooks.configs.flat.recommended,
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      globals: globals.browser,
    },
  },

  // Backend: Convex runtime (NOT browser)
  {
    files: ['convex/**/*.ts'],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommendedTypeChecked,
      ...convexPlugin.configs.recommended,
    ],
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-unused-vars': [
        'warn',
        { varsIgnorePattern: '^_', argsIgnorePattern: '^_' },
      ],
    },
  },
]);
```

**Key point:** No `globals.browser` for Convex — it doesn't run in a browser.

### Package.json Scripts

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

Using `eslint .` lints the entire project; each folder gets its own rules.

---

## Prettier Setup

Create `.prettierignore` to skip auto-generated files:

```
dist
convex/_generated
```

---

## Environment Variables Summary

- **Next.js** — `NEXT_PUBLIC_CONVEX_URL` via `process.env.NEXT_PUBLIC_CONVEX_URL`
- **Vite** — `VITE_CONVEX_URL` via `import.meta.env.VITE_CONVEX_URL`
- **Node.js** — `CONVEX_URL` via `process.env.CONVEX_URL`

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
