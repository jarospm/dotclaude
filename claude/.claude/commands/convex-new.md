---
description: Scaffold a new Convex project with TypeScript, ESLint, and Prettier
---

# Convex New Project

Scaffold a new Convex project in the **current directory** — backend-first, no frontend framework.

Uses the current directory name as the project name.

## What Gets Created

- `convex/` folder with:
  - `schema.ts` — Table validators, enums, derived types
  - `tasks.ts` — Sample CRUD functions with best practices
  - `README.md` — Backend patterns documentation
- `CLAUDE.md` — AI assistant guidance for the project
- TypeScript configuration
- ESLint with `@convex-dev/eslint-plugin` (flat config)
- Prettier as formatter
- Proper `.gitignore`

**Patterns demonstrated:**

- `Table()` utility from `convex-helpers/server`
- `literals()` for enum validators
- `pick()` and `partial()` for DRY patterns
- Proper `returns` validators using `Tasks.doc`, `Tasks._id`

## Prerequisites

Before starting, verify:

1. Node.js 18+ is installed (`node --version`)
2. npm is available (`npm --version`)
3. Current directory is empty or has no conflicting files (`package.json`, `convex/`)

If prerequisites fail, inform the user and stop.

---

## Step 1: Initialize npm

```bash
npm init -y
```

Then update `package.json` with proper configuration (use current directory name for the `name` field):

```json
{
  "name": "<current-directory-name>",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "convex dev",
    "deploy": "convex deploy",
    "lint": "eslint convex/",
    "lint:fix": "eslint convex/ --fix",
    "format": "prettier --write convex/",
    "format:check": "prettier --check convex/",
    "typecheck": "tsc -p convex --noEmit"
  }
}
```

---

## Step 2: Install Dependencies

```bash
npm install convex convex-helpers
npm install -D typescript eslint prettier @convex-dev/eslint-plugin typescript-eslint @eslint/js eslint-config-prettier
```

---

## Step 3: Create Configuration Files

### tsconfig.json

This is a minimal root config. Convex will generate its own `convex/tsconfig.json` with required settings.

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["convex/**/*"],
  "exclude": ["node_modules"]
}
```

### eslint.config.js

```javascript
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import convexPlugin from "@convex-dev/eslint-plugin";
import eslintConfigPrettier from "eslint-config-prettier";

export default [
  js.configs.recommended,
  ...tseslint.configs.recommended,
  ...convexPlugin.configs.recommended,
  eslintConfigPrettier,

  {
    files: ["convex/**/*.ts"],
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/no-unused-vars": [
        "warn",
        { varsIgnorePattern: "^_", argsIgnorePattern: "^_" },
      ],
    },
  },

  {
    ignores: ["convex/_generated/**", "node_modules/**", "dist/**"],
  },
];
```

### .prettierrc

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "arrowParens": "always"
}
```

### .prettierignore

```
convex/_generated
node_modules
dist
.env*
```

### .gitignore

```
node_modules/
dist/
.env.local
.env*.local
convex/_generated/

# Editor
.vscode/
.idea/
*.swp
*.swo
.DS_Store
```

---

## Step 4: Initialize Convex (User Action Required)

**PAUSE HERE** — Ask the user to run the Convex dev server:

```
Please run `npx convex dev` in your terminal.

This will:
1. Prompt for GitHub login (if not authenticated)
2. Create a new Convex project
3. Generate `convex/_generated/` files
4. Generate `convex/tsconfig.json`
5. Create `.env.local` with deployment URL
6. Enter watch mode

Keep the server running and let me know when it's ready.
```

**Wait for user confirmation before continuing.**

From this point on, assume the dev server is running. Never run `npx convex dev` yourself — always ask the user to manage the server.

---

## Step 5: Create Starter Files

### convex/schema.ts

Demonstrates:

- `Table()` utility for single source of truth
- `literals()` for concise enum validators
- Field ordering convention
- `Infer<>` for derived types

```typescript
import { defineSchema } from "convex/server";
import { v, Infer } from "convex/values";
import { Table } from "convex-helpers/server";
import { literals } from "convex-helpers/validators";

// =============================================================================
// Enums — Using literals() for concise union types
// =============================================================================

export const vPriority = literals("low", "medium", "high");
export const vStatus = literals("todo", "in_progress", "done");

// =============================================================================
// Tables — Single source of truth with built-in accessors
// =============================================================================
//
// Field ordering convention:
// 1. Identifiers (title, name, slug)
// 2. Foreign keys (projectId, userId)
// 3. Classification (priority, status, type)
// 4. Core content
// 5. Optional metadata
// 6. State flags
// 7. Timestamps (updatedAt)

export const Tasks = Table("tasks", {
  title: v.string(),
  priority: vPriority,
  status: vStatus,
  notes: v.optional(v.string()),
  updatedAt: v.number(),
});

// =============================================================================
// Derived Types — Use Infer<> instead of manual type definitions
// =============================================================================

export type Priority = Infer<typeof vPriority>;
export type Status = Infer<typeof vStatus>;
export type Task = Infer<typeof Tasks.doc>;

// =============================================================================
// Schema Definition
// =============================================================================

export default defineSchema({
  tasks: Tasks.table
    .index("by_status", ["status"])
    .index("by_priority", ["priority"]),
});
```

### convex/tasks.ts

Demonstrates:

- `Tasks._id` for ID validators
- `Tasks.doc` for full document returns
- `Tasks.withoutSystemFields` with `pick()` for args
- `partial()` for update mutations
- Explicit table names in db methods (Convex 1.31+)

```typescript
import { query, mutation } from "./_generated/server";
import { v } from "convex/values";
import { pick } from "convex-helpers";
import { partial } from "convex-helpers/validators";
import { Tasks, vStatus } from "./schema";

// List all tasks — return full documents using Tasks.doc
export const list = query({
  args: {},
  returns: v.array(Tasks.doc),
  handler: async (ctx) => {
    return await ctx.db.query("tasks").collect();
  },
});

// Get single task — nullable return
export const get = query({
  args: { id: Tasks._id },
  returns: v.union(Tasks.doc, v.null()),
  handler: async (ctx, { id }) => {
    return await ctx.db.get("tasks", id);
  },
});

// Create task — pick specific fields from withoutSystemFields
export const create = mutation({
  args: pick(Tasks.withoutSystemFields, ["title", "priority", "notes"]),
  returns: Tasks._id,
  handler: async (ctx, args) => {
    return await ctx.db.insert("tasks", {
      ...args,
      status: "todo",
      updatedAt: Date.now(),
    });
  },
});

// Update status — use enum validator directly
export const updateStatus = mutation({
  args: {
    id: Tasks._id,
    status: vStatus,
  },
  returns: v.null(),
  handler: async (ctx, { id, status }) => {
    await ctx.db.patch("tasks", id, { status, updatedAt: Date.now() });
    return null;
  },
});

// Update task — partial fields (all optional)
export const update = mutation({
  args: {
    id: Tasks._id,
    ...partial(pick(Tasks.withoutSystemFields, ["title", "priority", "notes"])),
  },
  returns: v.null(),
  handler: async (ctx, { id, ...updates }) => {
    const patch: Record<string, unknown> = { updatedAt: Date.now() };
    for (const [key, value] of Object.entries(updates)) {
      if (value !== undefined) patch[key] = value;
    }
    await ctx.db.patch("tasks", id, patch);
    return null;
  },
});

// Delete task
export const remove = mutation({
  args: { id: Tasks._id },
  returns: v.null(),
  handler: async (ctx, { id }) => {
    await ctx.db.delete("tasks", id);
    return null;
  },
});
```

### convex/README.md

Replace the auto-generated README with project-specific documentation:

````markdown
# Convex Backend

Backend for <project-name>.

## Schema

### Tables

**tasks** — Work items with priority and status tracking

- Indexed by: `status`, `priority`

### Enums

Defined using `literals()` from `convex-helpers/validators`:

- **Priority**: `low` | `medium` | `high`
- **Status**: `todo` | `in_progress` | `done`

### Field Ordering Convention

Fields in validators follow this order:

1. Identifiers (title, name, slug, url)
2. Foreign keys / relationships
3. Classification (type, priority, status)
4. Core content
5. Optional metadata
6. State and flags
7. Denormalized counts
8. Timestamps (updatedAt)

## Patterns

### Table Validators

Using `Table()` utility from `convex-helpers/server`:

```typescript
export const Tasks = Table("tasks", { ...fields });
```

Provides accessors:

- `Tasks.table` — for `defineSchema()`
- `Tasks.doc` — full document validator (v.object with all fields + system fields)
- `Tasks.withoutSystemFields` — user fields only (for inserts, args)
- `Tasks.withSystemFields` — user fields + `_id` and `_creationTime`
- `Tasks.systemFields` — just `_id` and `_creationTime`
- `Tasks._id` — ID validator for this table

Types derived using `Infer<typeof Tasks.doc>`.

### Function Args

Use `pick()` and `partial()` from `convex-helpers`:

```typescript
// Create — pick required fields
args: pick(Tasks.withoutSystemFields, ['title', 'priority']),

// Update — partial for optional fields
args: {
  id: Tasks._id,
  ...partial(pick(Tasks.withoutSystemFields, ['title', 'priority', 'notes'])),
},
```

### Function Naming

Minimal names — file provides context:

- CRUD: `create`, `update`, `get`, `remove`
- Qualified: `getByUrl`, `getBySlug`
- Lists: `list`, `listByProject`

### Return Values

- Create → `returns: Tasks._id`
- Update/patch → `returns: v.null()` (success is implicit)
- Get → `returns: v.union(Tasks.doc, v.null())`
- List → `returns: v.array(Tasks.doc)`
```

### CLAUDE.md (project root)

Create AI assistant guidance in the project root:

````markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Dev server:** `npm run dev` (starts Convex dev server)
- **Deploy:** `npm run deploy` (deploy to production)
- **Type check:** `npm run typecheck`
- **Lint:** `npm run lint`
- **Format:** `npm run format`
- **Format check:** `npm run format:check`

**After implementing changes**, always run:

```bash
npm run typecheck && npm run lint && npm run format
```
```

## Architecture

Convex backend-only project.

**File structure:**

- `convex/` – Convex backend (separate TypeScript project)
  - `_generated/` – Auto-generated types (don't edit)
  - `schema.ts` – Database schema with Table validators
  - `tasks.ts` – Task CRUD functions
  - `tsconfig.json` – Convex-specific TypeScript config
- `tsconfig.json` – Root TypeScript config
- `eslint.config.js` – ESLint flat config with Convex plugin
- `.prettierrc` – Prettier configuration

## Convex Backend

**Skill:** Always use the `convex-development` skill when writing or modifying Convex code.

See `convex/README.md` for schema details and patterns.

**Import ordering:**

```typescript
// 1. External packages
import { v, Infer } from "convex/values";
import { pick } from "convex-helpers";
import { partial } from "convex-helpers/validators";
import { Table } from "convex-helpers/server";

// 2. Generated files
import { mutation, query } from "./_generated/server";

// 3. Local files
import { Tasks, vStatus } from "./schema";
```

**Function naming:**

- Minimal names — file provides context: `tasks.create` not `createTask`
- CRUD: `create`, `update`, `get`, `remove`
- Qualified queries: `getByUrl`, `getBySlug`
- List queries: `list`, `listByProject`

**Schema patterns:**

- `Table()` utility from `convex-helpers/server` for single source of truth
- `literals()` for enum validators
- `pick()` to select fields for args
- `partial()` for optional update fields
- Types derived using `Infer<typeof Table.doc>`

**Mutation patterns:**

- `returns: v.null()` for update/patch operations
- `returns: Table._id` for create operations
- Always include `updatedAt: Date.now()` in mutations that modify data

---

## Step 6: Verify Setup

Run these commands to verify everything works:

```bash
npm run lint          # Should pass with no errors
npm run format:check  # Should pass
npm run typecheck     # Should pass
```

If all checks pass, the project is ready.

---

## Final Output

After completion, display:

```
✓ Project scaffolded successfully

Your dev server is already running and has synced the starter files.

Project structure:
  convex/
    schema.ts          # Database schema with Table validators
    tasks.ts           # Sample CRUD functions
    README.md          # Backend patterns documentation
  CLAUDE.md            # AI assistant guidance
  package.json
  tsconfig.json
  eslint.config.js
  .prettierrc

Commands:
  npm run dev          # Start dev server
  npm run lint         # Run ESLint
  npm run format       # Run Prettier
  npm run typecheck    # Type check convex/
  npm run deploy       # Deploy to production
```

---

## Notes

- This creates a **backend-only** project — no frontend
- The sample `tasks` table demonstrates best practices — modify or remove it
- **Dev server management:** Never run `npx convex dev` yourself — always ask the user to start/stop/restart the server
- Key patterns demonstrated:
  - `Table()` — single source of truth for validators
  - `literals()` — concise enum validators
  - `Tasks.doc` / `Tasks._id` — built-in accessors for returns
  - `pick()` + `partial()` — DRY args for create/update
  - `Infer<>` — derive types from validators
