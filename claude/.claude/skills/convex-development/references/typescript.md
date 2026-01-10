# Convex TypeScript Reference

TypeScript patterns and best practices for Convex development.

## Generated Types

Convex generates types in `convex/_generated/`:

```typescript
import { Id, Doc } from "./_generated/dataModel";
import { api, internal } from "./_generated/api";
import { query, mutation, action } from "./_generated/server";
import { internalQuery, internalMutation, internalAction } from "./_generated/server";
```

## Document ID Types

Use `Id<"tableName">` for strict typing:

```typescript
import { Id } from "./_generated/dataModel";

// Function parameter
function processUser(userId: Id<"users">) {
  // userId is strictly typed to users table
}

// In validators
args: {
  userId: v.id("users"),      // validates as Id<"users">
  messageId: v.id("messages"), // validates as Id<"messages">
}
```

**Don't use `string` for document IDs:**

```typescript
// Bad
function getUser(userId: string) { ... }

// Good
function getUser(userId: Id<"users">) { ... }
```

## Document Types

Use `Doc<"tableName">` for full document types:

```typescript
import { Doc } from "./_generated/dataModel";

type User = Doc<"users">;
// Includes _id, _creationTime, and all schema fields

function formatUser(user: Doc<"users">): string {
  return `${user.name} (${user._id})`;
}
```

## Extracting Types from Validators

### Using `Infer`

Extract TypeScript types from validators — single source of truth:

```typescript
import { v, Infer } from "convex/values";

const courseValidator = v.union(
  v.literal("appetizer"),
  v.literal("main"),
  v.literal("dessert")
);

// Extracts: "appetizer" | "main" | "dessert"
type Course = Infer<typeof courseValidator>;
```

### Using `ObjectType`

For plain objects of validators (not wrapped in `v.object`):

```typescript
import { v, ObjectType } from "convex/values";

const recipeArgs = {
  name: v.string(),
  course: v.union(v.literal("main"), v.literal("dessert")),
};

// Extracts: { name: string; course: "main" | "dessert" }
type RecipeArgs = ObjectType<typeof recipeArgs>;
```

### Using `WithoutSystemFields`

Strip `_id` and `_creationTime` — useful for insert data:

```typescript
import type { WithoutSystemFields } from "convex/server";
import type { Doc } from "./_generated/dataModel";

// Document without system fields (for inserts)
type NewRecipe = WithoutSystemFields<Doc<"recipes">>;

function createRecipeData(): NewRecipe {
  return {
    name: "Pasta",
    course: "main",
    // No _id or _creationTime needed
  };
}
```

## Record Types

When using `v.record()`, type the Record correctly:

```typescript
import { query } from "./_generated/server";
import { Id } from "./_generated/dataModel";
import { v } from "convex/values";

export const getUsernames = query({
  args: { userIds: v.array(v.id("users")) },
  returns: v.record(v.id("users"), v.string()),
  handler: async (ctx, args) => {
    const idToUsername: Record<Id<"users">, string> = {};

    for (const userId of args.userIds) {
      const user = await ctx.db.get(userId);
      if (user) {
        idToUsername[userId] = user.name;
      }
    }

    return idToUsername;
  },
});
```

## Array Types

Always declare arrays with explicit types:

```typescript
// Good
const users: Array<Doc<"users">> = [];
const ids: Array<Id<"users">> = [];

// Also good
const users: Doc<"users">[] = [];
```

## Discriminated Unions

Use `as const` for literal types in unions:

```typescript
// In handler code
const result = {
  kind: "success" as const,
  value: 42,
};

// Or with explicit type
type Result =
  | { kind: "success"; value: number }
  | { kind: "error"; message: string };

const result: Result = {
  kind: "success",
  value: 42,
};
```

## Same-File Function Calls

When calling a function defined in the same file, add type annotations:

```typescript
export const getUser = query({
  args: { userId: v.id("users") },
  returns: v.string(),
  handler: async (ctx, args) => {
    return "user name";
  },
});

export const processUsers = query({
  args: {},
  returns: v.null(),
  handler: async (ctx, args) => {
    // Add type annotation to avoid TypeScript circularity error
    const name: string = await ctx.runQuery(api.users.getUser, {
      userId: someId,
    });
    return null;
  },
});
```

## Return Type Patterns

### Nullable Returns

```typescript
export const maybeGetUser = query({
  args: { userId: v.id("users") },
  returns: v.union(
    v.object({
      _id: v.id("users"),
      _creationTime: v.number(),
      name: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db.get(args.userId);  // returns Doc | null
  },
});
```

### Void Returns

```typescript
export const doSomething = mutation({
  args: {},
  returns: v.null(),  // Use v.null() for void returns
  handler: async (ctx, args) => {
    // Do work...
    return null;  // Explicit null return
  },
});
```

## Node.js Types

Add `@types/node` to `package.json` when using Node.js modules:

```json
{
  "devDependencies": {
    "@types/node": "^20.0.0"
  }
}
```

## Helper Function Types

Type helper functions that work with Convex data:

```typescript
import { Doc, Id } from "./_generated/dataModel";
import { QueryCtx, MutationCtx } from "./_generated/server";

// Helper that takes context
async function getUserOrThrow(
  ctx: QueryCtx | MutationCtx,
  userId: Id<"users">
): Promise<Doc<"users">> {
  const user = await ctx.db.get(userId);
  if (!user) throw new Error("User not found");
  return user;
}

// Helper that transforms documents
function formatMessage(message: Doc<"messages">): {
  id: string;
  text: string;
  timestamp: number;
} {
  return {
    id: message._id,
    text: message.content,
    timestamp: message._creationTime,
  };
}
```

## Common Type Errors

### "Type 'string' is not assignable to type 'Id<...>'"

```typescript
// Wrong: string literal
const userId = "abc123";
await ctx.db.get(userId);  // Error!

// Right: properly typed ID from args or db
const userId = args.userId;  // Already Id<"users"> from validator
await ctx.db.get(userId);    // Works
```

### "Argument of type '...' is not assignable to parameter"

```typescript
// Usually means wrong function reference
// Wrong:
await ctx.runQuery(myQuery, args);  // Don't pass function directly

// Right:
await ctx.runQuery(api.module.myQuery, args);  // Use function reference
```

## tsconfig.json for Convex

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["ES2021", "dom"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["./**/*"],
  "exclude": ["./_generated"]
}
```
