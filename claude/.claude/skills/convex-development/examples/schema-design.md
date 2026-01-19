# Example: Schema Design Patterns

Best practices for designing Convex schemas with reusable validators.

## Key Patterns

- Exported validators with `v.object()` wrapper for derivation
- Enum validators for constrained values
- Field ordering convention for consistency
- Format guidance in comments
- Denormalized counts for efficient queries
- Derived types using `Infer<>`

## Complete Schema

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v, Infer } from "convex/values";

// =============================================================================
// Enums — Union types for constrained values
// =============================================================================

/** Task priority levels */
export const vPriority = v.union(
  v.literal("low"),
  v.literal("medium"),
  v.literal("high")
);

/** Task workflow status */
export const vStatus = v.union(
  v.literal("todo"),
  v.literal("in_progress"),
  v.literal("done"),
  v.literal("archived")
);

// =============================================================================
// Entity Validators — Wrapped in v.object() to enable derivation methods
// =============================================================================
//
// Use v.object() wrapper (not inline objects) because it enables:
// - .fields — extract plain object for defineTable()
// - .pick() — select specific fields for partial queries
// - .partial() — make all fields optional (for patches)
// - .extend() — add fields (e.g., system fields for returns)

/**
 * Project — container for tasks
 *
 * Field ordering convention:
 * 1. Identifiers (name, slug, title)
 * 2. Foreign keys / relationships
 * 3. Classification (type, category)
 * 4. Core content
 * 5. Optional metadata
 * 6. State and flags
 * 7. Denormalized counts
 * 8. Timestamps (updatedAt)
 * 9. Embedding (always last)
 */
export const vProject = v.object({
  name: v.string(), // 1-3 words
  slug: v.string(), // URL-safe identifier
  description: v.optional(v.string()), // 1-2 sentences
  isArchived: v.boolean(),
  openTaskCount: v.number(),
  completedTaskCount: v.number(),
  updatedAt: v.number(),
});

/** Task — work item within a project */
export const vTask = v.object({
  projectId: v.id("projects"),
  priority: vPriority,
  title: v.string(), // single line, under 100 chars
  notes: v.optional(v.string()), // multi-line if needed
  dueDate: v.optional(v.number()),
  assigneeId: v.optional(v.id("users")),
  status: vStatus,
  updatedAt: v.number(),
});

// =============================================================================
// Derived Types — Use Infer<> instead of manual type definitions
// =============================================================================

export type Priority = Infer<typeof vPriority>;
export type Status = Infer<typeof vStatus>;
export type Project = Infer<typeof vProject>;
export type Task = Infer<typeof vTask>;

// =============================================================================
// Schema Definition — use .fields to extract plain object from v.object()
// =============================================================================

export default defineSchema({
  projects: defineTable(vProject.fields).index("by_slug", ["slug"]),

  tasks: defineTable(vTask.fields)
    .index("by_projectId", ["projectId"])
    .index("by_status", ["status"])
    .index("by_priority", ["priority"]),
});
```

## Pattern Explanations

### 1. Validator Naming

Prefix with `v` to mirror the `v.*` import:

```typescript
export const vStatus = v.union(...);   // enum
export const vTask = v.object({...});  // entity
```

### 2. v.object() Wrapper

Enables derivation methods in functions:

```typescript
args: vTask.fields,                    // all fields
args: vTask.pick('title').fields,      // just { title }
args: { patch: vTask.partial() },      // all optional for updates
returns: vTask.extend({                // add system fields
  _id: v.id('tasks'),
  _creationTime: v.number(),
}),
```

### 3. Field Ordering Convention

Consistent ordering across all validators:

1. **Identifiers** — `name`, `slug`, `title`
2. **Foreign keys** — `projectId`, `assigneeId`
3. **Classification** — `type`, `priority`
4. **Core content** — main data fields
5. **Optional metadata** — supplementary info
6. **State and flags** — `status`, `isArchived`
7. **Denormalized counts** — `openTaskCount`
8. **Timestamps** — `updatedAt`
9. **Embedding** — always last (if present)

### 4. Format Guidance Comments

Document expected formats for consistent data:

```typescript
title: v.string(),    // single line, under 100 chars
name: v.string(),     // 1-3 words
description: v.optional(v.string()), // 1-2 sentences
```

### 5. Denormalized Counts

Store counts on parent for fast queries:

```typescript
export const vProject = v.object({
  openTaskCount: v.number(), // tasks with status != 'done'
  completedTaskCount: v.number(), // tasks with status == 'done'
});
```

**Benefits:** Fast list queries, derive state (`hasWork = openTaskCount > 0`)

**Trade-off:** Update counts when child status changes

### 6. Derived Types

Use `Infer<>` instead of manual type definitions:

```typescript
// CORRECT — single source of truth
export type Task = Infer<typeof vTask>;

// AVOID — duplicates the definition
export type Task = {
  projectId: Id<"projects">;
  title: string;
  // ... easy to get out of sync
};
```

## Using Validators in Functions

```typescript
// convex/tasks.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";
import { vTask, vStatus } from "./schema";

export const create = mutation({
  args: vTask.fields,
  returns: v.id("tasks"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("tasks", args);
  },
});

export const updateStatus = mutation({
  args: {
    id: v.id("tasks"),
    status: vStatus,
  },
  returns: v.null(),
  handler: async (ctx, { id, status }) => {
    await ctx.db.patch("tasks", id, { status, updatedAt: Date.now() });
    return null;
  },
});

export const listByProject = query({
  args: vTask.pick("projectId").fields,
  returns: v.array(
    vTask.extend({
      _id: v.id("tasks"),
      _creationTime: v.number(),
    })
  ),
  handler: async (ctx, { projectId }) => {
    return await ctx.db
      .query("tasks")
      .withIndex("by_projectId", (q) => q.eq("projectId", projectId))
      .collect();
  },
});
```

---

## Alternative: Table Utility (Recommended)

The same schema using `convex-helpers` Table utility for cleaner patterns:

### Schema with Table()

```typescript
// convex/schema.ts
import { defineSchema } from "convex/server";
import { v, Infer } from "convex/values";
import { Table } from "convex-helpers/server";
import { literals } from "convex-helpers/validators";

// =============================================================================
// Enums — Using literals() instead of v.union(v.literal(...), ...)
// =============================================================================

export const vPriority = literals("low", "medium", "high");
export const vStatus = literals("todo", "in_progress", "done", "archived");

// =============================================================================
// Tables — Single source of truth with built-in accessors
// =============================================================================

export const Projects = Table("projects", {
  name: v.string(),              // 1-3 words
  slug: v.string(),              // URL-safe identifier
  description: v.optional(v.string()), // 1-2 sentences
  isArchived: v.boolean(),
  openTaskCount: v.number(),
  completedTaskCount: v.number(),
  updatedAt: v.number(),
});

export const Tasks = Table("tasks", {
  projectId: v.id("projects"),
  priority: vPriority,
  title: v.string(),             // single line, under 100 chars
  notes: v.optional(v.string()), // multi-line if needed
  dueDate: v.optional(v.number()),
  assigneeId: v.optional(v.id("users")),
  status: vStatus,
  updatedAt: v.number(),
});

// =============================================================================
// Derived Types
// =============================================================================

export type Priority = Infer<typeof vPriority>;
export type Status = Infer<typeof vStatus>;
export type Project = Infer<typeof Projects.doc>;
export type Task = Infer<typeof Tasks.doc>;

// =============================================================================
// Schema Definition
// =============================================================================

export default defineSchema({
  projects: Projects.table.index("by_slug", ["slug"]),

  tasks: Tasks.table
    .index("by_projectId", ["projectId"])
    .index("by_status", ["status"])
    .index("by_priority", ["priority"]),
});
```

### Functions with Table Validators

```typescript
// convex/tasks.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";
import { pick } from "convex-helpers";
import { partial } from "convex-helpers/validators";
import { Tasks, vStatus } from "./schema";

// Create — pick specific fields, no manual extension needed
export const create = mutation({
  args: pick(Tasks.withoutSystemFields, [
    "projectId",
    "priority",
    "title",
    "notes",
    "dueDate",
    "assigneeId",
  ]),
  returns: Tasks._id,  // Built-in ID validator
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

// List by project — return full documents (no manual extension!)
export const listByProject = query({
  args: { projectId: v.id("projects") },
  returns: v.array(Tasks.doc),  // Built-in doc validator
  handler: async (ctx, { projectId }) => {
    return await ctx.db
      .query("tasks")
      .withIndex("by_projectId", (q) => q.eq("projectId", projectId))
      .collect();
  },
});

// Update — partial fields with pick + partial combo
export const update = mutation({
  args: {
    id: Tasks._id,
    ...partial(pick(Tasks.withoutSystemFields, [
      "title",
      "notes",
      "priority",
      "dueDate",
      "assigneeId",
    ])),
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
```

### Key Differences from v.object() Approach

| Pattern | v.object() + defineTable() | Table() utility |
|---------|---------------------------|-----------------|
| ID validator | `v.id("tasks")` | `Tasks._id` |
| Full doc return | `vTask.extend({ _id: ..., _creationTime: ... })` | `Tasks.doc` |
| Fields for insert | `vTask.fields` | `Tasks.withoutSystemFields` |
| Table definition | `defineTable(vTask.fields)` | `Tasks.table` |
| Enum validators | `v.union(v.literal(...), ...)` | `literals(...)` |

**Benefits of Table():**
- No manual system field extension
- Built-in accessors for all common patterns
- Cleaner function signatures
- Single source of truth per table
