# Convex Validators Reference

Validators define the shape of function arguments, return values, and database schemas.

## Why Validation Matters

**Security is the primary reason.** Without argument validation, malicious users can call public functions with unexpected arguments. TypeScript types aren't present at runtime.

```typescript
// DANGEROUS: No validation
export const send = mutation({
  handler: async (ctx, args) => {
    // args could be anything!
  },
});

// SAFE: With validation
export const send = mutation({
  args: { body: v.string(), author: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // args.body and args.author are guaranteed strings
  },
});
```

**Rules:**
- Always validate **all public functions** in production
- For internal functions, validation is optional but still useful
- Even `args: {}` helps — TypeScript will error if clients pass unexpected arguments

## Import

```typescript
import { v } from "convex/values";
```

## Primitive Validators

| Validator | TypeScript Type | Example | Notes |
|-----------|-----------------|---------|-------|
| `v.string()` | `string` | `"abc"` | UTF-8, max 1MB |
| `v.number()` | `number` | `3.14` | IEEE-754 float64 |
| `v.boolean()` | `boolean` | `true` | |
| `v.null()` | `null` | `null` | Use instead of undefined |
| `v.int64()` | `bigint` | `3n` | -2^63 to 2^63-1 |
| `v.bytes()` | `ArrayBuffer` | `new ArrayBuffer(8)` | Max 1MB |

**Important:** JavaScript's `undefined` is NOT a valid Convex value. Use `null` instead.

## Document ID Validator

```typescript
v.id("tableName")  // References a document in the specified table
```

Example:
```typescript
args: {
  userId: v.id("users"),
  messageId: v.id("messages"),
}
```

## Compound Validators

### Arrays

```typescript
v.array(v.string())              // string[]
v.array(v.id("users"))           // Id<"users">[]
v.array(v.object({ ... }))       // object array
```

**Limit:** Arrays can have at most 8192 elements.

### Objects

```typescript
v.object({
  name: v.string(),
  age: v.number(),
  email: v.optional(v.string()),  // optional field
})
```

**Rules:**
- Max 1024 fields
- Field names must be non-empty
- Field names must be ASCII-only
- Field names cannot start with `$` or `_`

**Emoji/Unicode handling:** Object field names cannot contain non-ASCII characters. If you need emoji or unicode as keys, remap them to ASCII codes first:

```typescript
// WRONG: emoji as field name
{ "🔥": "hot" }  // Will fail!

// CORRECT: remap to ASCII
{ "emoji_fire": "hot" }
```

### Records (Dynamic Keys)

```typescript
v.record(v.string(), v.number())           // Record<string, number>
v.record(v.id("users"), v.string())        // Record<Id<"users">, string>
```

**Rules:**
- Keys must be ASCII characters only
- Keys must be non-empty
- Keys cannot start with `$` or `_`

### Optional Fields

```typescript
v.optional(v.string())   // string | undefined
v.optional(v.id("users")) // Id<"users"> | undefined
```

Use for fields that may not be present.

### Nullable (Shorthand)

```typescript
v.nullable(v.string())  // equivalent to v.union(v.string(), v.null())
```

### Any Type

```typescript
v.any()  // Accepts any valid Convex value
```

Use sparingly — prefer specific validators for type safety.

## Union and Literal Types

### Simple Union

```typescript
v.union(v.string(), v.number())  // string | number
```

### Discriminated Union (Tagged Union)

```typescript
v.union(
  v.object({
    kind: v.literal("error"),
    errorMessage: v.string(),
  }),
  v.object({
    kind: v.literal("success"),
    value: v.number(),
  })
)
```

### Literal Values

```typescript
v.literal("pending")    // exactly "pending"
v.literal("active")     // exactly "active"
```

## Complete Example

```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const createOrder = mutation({
  args: {
    userId: v.id("users"),
    items: v.array(
      v.object({
        productId: v.id("products"),
        quantity: v.number(),
        notes: v.optional(v.string()),
      })
    ),
    status: v.union(
      v.literal("pending"),
      v.literal("confirmed"),
      v.literal("shipped")
    ),
    metadata: v.optional(v.record(v.string(), v.string())),
  },
  returns: v.id("orders"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("orders", {
      userId: args.userId,
      items: args.items,
      status: args.status,
      metadata: args.metadata,
    });
  },
});
```

## Reusing Validators (DRY Patterns)

### Export Wrapped Validators from Schema

Use `v.object()` to enable `.pick()`, `.omit()`, `.partial()`, `.extend()` methods:

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

// Naming convention: "v" prefix mirrors the v.* import
export const vCourse = v.union(
  v.literal("appetizer"),
  v.literal("main"),
  v.literal("dessert")
);

// Wrapped v.object() enables derivation methods
export const vRecipe = v.object({
  name: v.string(),
  course: vCourse,
  ingredients: v.array(v.string()),
  steps: v.array(v.string()),
});

export default defineSchema({
  recipes: defineTable(vRecipe.fields).index("by_course", ["course"]),
});
```

### Using Derived Validators in Functions

```typescript
// convex/recipes.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";
import { vRecipe } from "./schema";

// Create: use all fields
export const create = mutation({
  args: vRecipe.fields,  // { name, course, ingredients, steps }
  returns: v.id("recipes"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("recipes", args);
  },
});

// Query by specific field: use .pick()
export const listByCourse = query({
  args: vRecipe.pick("course").fields,  // { course }
  handler: async (ctx, args) => {
    return await ctx.db
      .query("recipes")
      .withIndex("by_course", (q) => q.eq("course", args.course))
      .collect();
  },
});

// Update: use .partial() for optional fields
export const update = mutation({
  args: {
    id: v.id("recipes"),
    patch: vRecipe.partial(),  // All fields optional
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.patch(args.id, args.patch);
    return null;
  },
});
```

### Validator Derivation Methods

```typescript
const userValidator = v.object({
  name: v.string(),
  email: v.string(),
  status: v.union(v.literal("active"), v.literal("inactive")),
  profileUrl: v.optional(v.string()),
});

// Pick specific fields
userValidator.pick("name", "profileUrl")

// Omit specific fields
userValidator.omit("status", "profileUrl")

// Make all fields optional (for patches)
userValidator.partial()

// Add new fields
userValidator.extend({
  _id: v.id("users"),
  _creationTime: v.number(),
})

// Extract plain object for args: or defineTable()
userValidator.fields
```

### Schema Introspection

Access validators directly through the schema object:

```typescript
import schema from "./schema";
import { query } from "./_generated/server";

// Get the full table validator
const recipesValidator = schema.tables.recipes.validator;

// Access individual field validators
const courseValidator = recipesValidator.fields.course;

export const listByCourse = query({
  args: { course: courseValidator },
  handler: async (ctx, args) => {
    // ...
  },
});
```

## convex-helpers Utilities

The [`convex-helpers`](https://github.com/get-convex/convex-helpers) library provides additional utilities:

```typescript
import { partial } from "convex-helpers/server/validators";
import { doc, typedV } from "convex-helpers/validators";
import schema from "./schema";

// Type-safe v.id() that checks table names
export const vv = typedV(schema);
vv.id("reciepes")  // TypeScript error: typo!

// Full document validator including system fields
export const gradeRecipe = mutation({
  args: { recipe: doc(schema, "recipes") },
  handler: async (ctx, args) => {
    console.log(args.recipe._id, args.recipe.course);
  },
});
```

## Deprecated

- `v.bigint()` — Use `v.int64()` instead
- `v.map()` and `v.set()` — Not supported, use `v.record()` or `v.array()`
