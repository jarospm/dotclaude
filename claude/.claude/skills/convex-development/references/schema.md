# Convex Schema Reference

Schemas define your database structure in `convex/schema.ts`.

## Basic Schema

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
  }),

  messages: defineTable({
    content: v.string(),
    authorId: v.id("users"),
  }),
});
```

## System Fields

Every document automatically has:
- `_id` — Unique document ID (`v.id("tableName")`)
- `_creationTime` — Timestamp in milliseconds (`v.number()`)

**Do not include these in your schema definition.**

## Indexes

Indexes enable efficient queries. Without an index, queries scan the entire table.

### Defining Indexes

```typescript
export default defineSchema({
  messages: defineTable({
    channelId: v.id("channels"),
    authorId: v.id("users"),
    content: v.string(),
  })
    .index("by_channel", ["channelId"])
    .index("by_author", ["authorId"])
    .index("by_channel_and_author", ["channelId", "authorId"]),
});
```

### Index Naming Convention

Include all indexed fields in the name:
- Single field: `by_fieldName`
- Multiple fields: `by_field1_and_field2`

### Index Field Order Matters

Fields must be queried in the order they're defined:

```typescript
// Index: ["channelId", "authorId"]

// Valid queries:
.withIndex("by_channel_and_author", q => q.eq("channelId", id))
.withIndex("by_channel_and_author", q => q.eq("channelId", id).eq("authorId", aid))

// Invalid (can't skip channelId):
.withIndex("by_channel_and_author", q => q.eq("authorId", aid))  // Won't work!
```

### Built-in Indexes

Every table has these indexes automatically:
- `by_id` — Index on `_id`
- `by_creation_time` — Index on `_creationTime`

**Critical rules:**
- NEVER add `by_id` or `by_creation_time` to your schema — they're automatic and will cause an error
- NEVER include `_creationTime` as the last column in custom indexes — Convex automatically appends it to ALL indexes
- Index definitions MUST be nonempty

```typescript
// WRONG - causes errors:
.index("by_id", ["_id"])                              // Already exists!
.index("by_creation_time", ["_creationTime"])         // Already exists!
.index("by_author_and_time", ["author", "_creationTime"])  // _creationTime auto-appended!
.index("by_creation_time", [])                        // Empty index!

// CORRECT:
.index("by_author", ["author"])  // Convex adds _creationTime automatically
```

### Using the Built-in `by_creation_time` Index

```typescript
// Get messages from the last hour
const recentMessages = await ctx.db
  .query("messages")
  .withIndex("by_creation_time", (q) =>
    q.gt("_creationTime", Date.now() - 60 * 60 * 1000)
  )
  .collect();

// Get all messages in descending order (newest first)
const allMessages = await ctx.db
  .query("messages")
  .order("desc")
  .collect();
```

## Search Indexes

For full-text search capabilities:

```typescript
export default defineSchema({
  messages: defineTable({
    content: v.string(),
    channelId: v.id("channels"),
  })
    .index("by_channel", ["channelId"])
    .searchIndex("search_content", {
      searchField: "content",
      filterFields: ["channelId"],
    }),
});
```

### Using Search Indexes

```typescript
const results = await ctx.db
  .query("messages")
  .withSearchIndex("search_content", (q) =>
    q.search("content", "hello world").eq("channelId", channelId)
  )
  .take(10);
```

### Nested Field Paths

You can specify search and filter fields on nested documents using dot notation:

```typescript
export default defineSchema({
  products: defineTable({
    name: v.string(),
    properties: v.object({
      category: v.string(),
      description: v.string(),
    }),
  }).searchIndex("search_products", {
    searchField: "properties.description",
    filterFields: ["properties.category"],
  }),
});
```

## Discriminated Unions in Schema

```typescript
export default defineSchema({
  results: defineTable(
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
  ),
});
```

## Field Ordering Convention

For consistency across validators, order fields as follows:

1. **Identifiers** — `url`, `name`, `slug`, `title`
2. **Foreign keys** — `articleId`, `categoryIds`
3. **Classification** — `type`, `segmentIndex`
4. **Core content** — main data fields
5. **Optional metadata** — supplementary info
6. **State and flags** — `status`, `isHighlight`
7. **Denormalized counts** — `draftItemCount`, `itemCount`
8. **Timestamps** — `updatedAt`
9. **Embedding** — always last

## Format Guidance Comments

Document expected formats for text fields to ensure consistent data:

```typescript
export const vArticle = v.object({
  title: v.string(),           // single line
  summary: v.string(),         // 1-2 sentence TL;DR
  keyPoints: v.array(v.string()), // 3-7 bullets, each under 100 chars
  content: v.string(),         // 1-2 sentences, under 200 chars
});
```

**Why:** Ensures consistent data generation (especially with AI) and predictable UI rendering.

## Denormalized Counts Pattern

Store counts on parent documents for efficient queries:

```typescript
export const vArticle = v.object({
  // ... other fields
  draftItemCount: v.number(),     // count of items with status='draft'
  publishedItemCount: v.number(), // count of items with status='published'
});
```

**Benefits:**
- Fast queries without aggregation
- Derive state from counts: `needsReview = draftItemCount > 0`
- Display counts in lists without joins

**Trade-off:** Update counts in mutations when child status changes.

## Reusable Validators Pattern

For larger projects, export validators separately for reuse across functions.
See `references/validators.md` for full details on derivation methods.
See `examples/schema-design.md` for a comprehensive real-world example.

### Two Approaches

**1. v.object() + defineTable() (vanilla Convex)**
- Uses `v.object()` wrapper with `.fields`, `.pick()`, `.partial()`, `.extend()`
- Requires manual extension for system fields in return validators

**2. Table() utility (convex-helpers) — Recommended**
- Single definition provides all accessors automatically
- Built-in `._id`, `.doc`, `.withoutSystemFields`, `.table`
- No manual extension needed

### Naming Convention

Use `vXxx` prefix for exported validators (mirrors the `v.*` import):

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v, Infer } from "convex/values";

// =============================================================================
// Exported Validators — Single source of truth
// =============================================================================

/** Status enum validator */
export const vStatus = v.union(
  v.literal("pending"),
  v.literal("active"),
  v.literal("completed"),
);

/** Task fields validator — wrap in v.object() to enable .pick()/.partial() */
export const vTask = v.object({
  title: v.string(),
  status: vStatus,
  assigneeId: v.optional(v.id("users")),
  dueDate: v.optional(v.number()),
});

// =============================================================================
// Derived Types — Use Infer<> instead of manual type definitions
// =============================================================================

export type Status = Infer<typeof vStatus>;
export type Task = Infer<typeof vTask>;

// =============================================================================
// Schema Definition — use .fields to extract plain object from v.object()
// =============================================================================

export default defineSchema({
  tasks: defineTable(vTask.fields)
    .index("by_status", ["status"])
    .index("by_assignee", ["assigneeId"]),
});
```

### Using Validators in Functions

```typescript
// convex/tasks.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";
import { vTask } from "./schema";

export const create = mutation({
  args: vTask.fields,  // All task fields
  returns: v.id("tasks"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("tasks", args);
  },
});

export const updateStatus = mutation({
  args: {
    id: v.id("tasks"),
    ...vTask.pick("status").fields,  // Just { status }
  },
  returns: v.null(),
  handler: async (ctx, { id, status }) => {
    await ctx.db.patch("tasks", id, { status });
    return null;
  },
});
```

## Table Utility Approach (Recommended)

Using `convex-helpers` Table utility for single-source-of-truth definitions:

```typescript
// convex/schema.ts
import { defineSchema } from "convex/server";
import { v, Infer } from "convex/values";
import { Table } from "convex-helpers/server";
import { literals } from "convex-helpers/validators";

// =============================================================================
// Enums — Using literals() for concise union types
// =============================================================================

export const vStatus = literals("pending", "active", "completed");
export const vPriority = literals("low", "medium", "high");

// =============================================================================
// Tables — Using Table() for single source of truth
// =============================================================================

export const Tasks = Table("tasks", {
  title: v.string(),
  projectId: v.id("projects"),
  priority: vPriority,
  status: vStatus,
  assigneeId: v.optional(v.id("users")),
  dueDate: v.optional(v.number()),
  updatedAt: v.number(),
});

export const Projects = Table("projects", {
  name: v.string(),
  slug: v.string(),
  isArchived: v.boolean(),
  taskCount: v.number(),
  updatedAt: v.number(),
});

// =============================================================================
// Derived Types — Use Infer<typeof Table.doc>
// =============================================================================

export type Status = Infer<typeof vStatus>;
export type Task = Infer<typeof Tasks.doc>;
export type Project = Infer<typeof Projects.doc>;

// =============================================================================
// Schema Definition — Use Table.table
// =============================================================================

export default defineSchema({
  tasks: Tasks.table
    .index("by_projectId", ["projectId"])
    .index("by_status", ["status"]),

  projects: Projects.table
    .index("by_slug", ["slug"]),
});
```

**Table accessors:**

- `Tasks._id` — `v.id("tasks")` for args
- `Tasks.doc` — Full document validator with `_id`, `_creationTime`
- `Tasks.withoutSystemFields` — Fields only, for `pick()` and inserts
- `Tasks.table` — TableDefinition for `defineSchema()`

**Using in functions:**

```typescript
import { pick } from "convex-helpers";
import { partial } from "convex-helpers/validators";
import { Tasks } from "./schema";

// Create: pick fields needed from caller
args: pick(Tasks.withoutSystemFields, ["title", "projectId", "priority"])

// Get by ID
args: { id: Tasks._id }
returns: v.union(Tasks.doc, v.null())

// Update: partial fields
args: {
  id: Tasks._id,
  ...partial(pick(Tasks.withoutSystemFields, ["title", "status", "priority"]))
}
```

## Complete Example (Inline)

Simple approach for smaller projects:

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    avatarId: v.optional(v.id("_storage")),
  })
    .index("by_email", ["email"]),

  channels: defineTable({
    name: v.string(),
    isPrivate: v.boolean(),
  }),

  messages: defineTable({
    channelId: v.id("channels"),
    authorId: v.optional(v.id("users")),  // optional for system messages
    content: v.string(),
  })
    .index("by_channel", ["channelId"])
    .searchIndex("search_content", {
      searchField: "content",
      filterFields: ["channelId"],
    }),
});
```
