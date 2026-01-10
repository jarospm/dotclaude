# Convex ESLint Reference

ESLint rules for Convex functions enforce best practices and catch common mistakes.

## Installation

```bash
npm i @convex-dev/eslint-plugin --save-dev
```

## Configuration (ESLint 9 Flat Config)

```javascript
// eslint.config.js
import { defineConfig } from "eslint/config";
import convexPlugin from "@convex-dev/eslint-plugin";

export default defineConfig([
  // Other configurations...

  ...convexPlugin.configs.recommended,
]);
```

## Rules

### no-old-registered-function-syntax

Prefer object syntax for registered functions. **Auto-fixable.**

```typescript
// ✅ Correct
export const list = query({
  handler: async (ctx) => {
    return await ctx.db.query("messages").collect();
  },
});

// ❌ Wrong — old syntax
export const list = query(async (ctx) => {
  return await ctx.db.query("messages").collect();
});
```

### require-argument-validators

Require argument validators for all functions. **Auto-fixable.**

```typescript
// ✅ Correct
export const list = query({
  args: {},
  handler: async (ctx) => { ... },
});

// ✅ Correct — with args
export const get = query({
  args: { id: v.id("messages") },
  handler: async (ctx, { id }) => { ... },
});

// ❌ Wrong — missing args
export const list = query({
  handler: async (ctx) => { ... },
});
```

### explicit-table-ids

Require explicit table names in database operations. **Auto-fixable.**

Since Convex 1.31.0, include table name as first argument to db methods.

```typescript
const messageId: Id<"messages"> = ...;

// ✅ Correct — explicit table name
await ctx.db.get("messages", messageId);
await ctx.db.patch("messages", messageId, { text: "updated" });
await ctx.db.replace("messages", messageId, { text: "replaced", author: "Alice" });
await ctx.db.delete("messages", messageId);

// ❌ Wrong — implicit table (deprecated)
await ctx.db.get(messageId);
await ctx.db.patch(messageId, { text: "updated" });
```

**Alternative migration:** Use the codemod CLI instead of ESLint:
```bash
npx @convex-dev/codemod@latest explicit-ids
```

### import-wrong-runtime

Prevent Convex runtime files from importing Node runtime files (`"use node"`).

```typescript
// In a file without "use node":

// ✅ Correct — importing from non-Node file
import { someFunction } from "./helpers";

// ❌ Wrong — importing from Node file
import { someFunction } from "./nodeHelpers"; // has "use node"
```

---

## Additional Best Practices

### Await All Promises

Prevent floating promises in server functions:

```javascript
// eslint.config.js or .eslintrc.js rules section
"@typescript-eslint/no-floating-promises": "error",
```

Requires `@typescript-eslint/eslint-plugin`:
```bash
npm i -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### Enforce Custom Functions

When using custom function wrappers, prevent importing raw functions:

```javascript
// eslint.config.js rules section
"no-restricted-imports": [
  "error",
  {
    patterns: [
      {
        group: ["*/_generated/server"],
        importNames: ["query", "mutation", "action"],
        message: "Use functions.ts for query, mutation, action",
      },
    ],
  },
],
```

Then in `convex/functions.ts`:

```typescript
/* eslint-disable no-restricted-imports */
import {
  action as actionRaw,
  mutation as mutationRaw,
  query as queryRaw,
} from "./_generated/server";
/* eslint-enable no-restricted-imports */

import { customQuery, customMutation, customAction } from "convex-helpers/server/customFunctions";

// Your custom wrappers with auth checks, logging, etc.
export const query = customQuery(queryRaw, yourMiddleware);
export const mutation = customMutation(mutationRaw, yourMiddleware);
export const action = customAction(actionRaw, yourMiddleware);
```

---

## Complete ESLint Config Example

For a React/Vite + Convex project:

```javascript
// eslint.config.js
import { defineConfig } from "eslint/config";
import convexPlugin from "@convex-dev/eslint-plugin";
import tseslint from "typescript-eslint";

export default defineConfig([
  ...tseslint.configs.recommended,
  ...convexPlugin.configs.recommended,

  {
    files: ["**/*.ts", "**/*.tsx"],
    rules: {
      // Await promises
      "@typescript-eslint/no-floating-promises": "error",

      // Allow unused vars prefixed with _
      "@typescript-eslint/no-unused-vars": [
        "warn",
        { varsIgnorePattern: "^_", argsIgnorePattern: "^_" },
      ],

      // Enforce custom functions
      "no-restricted-imports": [
        "error",
        {
          patterns: [
            {
              group: ["*/_generated/server"],
              importNames: ["query", "mutation", "action"],
              message: "Use functions.ts for query, mutation, action",
            },
          ],
        },
      ],
    },
  },

  {
    ignores: ["convex/_generated/**", "node_modules/**", "dist/**"],
  },
]);
```

Install dependencies:
```bash
npm i -D eslint @convex-dev/eslint-plugin typescript-eslint @typescript-eslint/parser
```

---

## Next.js Integration

For `next lint` to include the `convex/` directory:

```typescript
// next.config.ts
const nextConfig: NextConfig = {
  eslint: {
    dirs: ["pages", "app", "components", "lib", "src", "convex"],
  },
};
```

---

## Custom Convex Directory

If your Convex folder is not `convex/`:

```javascript
// eslint.config.js
import { defineConfig } from "eslint/config";
import convexPlugin from "@convex-dev/eslint-plugin";

const recommendedConfig = convexPlugin.configs.recommended[0];
const recommendedRules = recommendedConfig.rules;

export default defineConfig([
  {
    files: ["**/src/backend/**/*.ts"],  // Your custom path
    plugins: {
      "@convex-dev": convexPlugin,
    },
    rules: recommendedRules,
  },
]);
```

---

## Package.json Scripts

```json
{
  "scripts": {
    "lint": "eslint convex/",
    "lint:fix": "eslint convex/ --fix"
  }
}
```
