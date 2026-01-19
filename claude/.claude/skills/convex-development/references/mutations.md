# Convex Mutations Reference

Mutations write data to the database. They run as transactions.

## Basic Mutation

```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const createUser = mutation({
  args: {
    name: v.string(),
    email: v.string(),
  },
  returns: v.id("users"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("users", {
      name: args.name,
      email: args.email,
    });
  },
});
```

## Internal Mutation

```typescript
import { internalMutation } from "./_generated/server";

export const updateStats = internalMutation({
  args: { userId: v.id("users") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Only callable from other Convex functions
    return null;
  },
});
```

## Database Operations

### Insert

```typescript
const docId = await ctx.db.insert("tableName", {
  field1: "value",
  field2: 123,
});
```

### Patch (Shallow Merge)

Updates specific fields, keeps others unchanged:

```typescript
await ctx.db.patch("users", documentId, {
  status: "active",
  updatedAt: Date.now(),
});
// Other fields remain unchanged
```

**Throws error if document doesn't exist.**

### Replace (Full Replace)

Replaces entire document (except system fields):

```typescript
await ctx.db.replace("users", documentId, {
  name: "New Name",
  email: "new@email.com",
  // Must include ALL fields
});
```

**Throws error if document doesn't exist.**

### Delete

```typescript
await ctx.db.delete("users", documentId);
```

**To delete multiple documents:**

```typescript
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .collect();

for (const message of messages) {
  await ctx.db.delete("messages", message._id);
}
```

**Note:** Convex queries do NOT support `.delete()` method.

## Transactions

Mutations are atomic transactions:
- All operations succeed or all fail
- No partial updates
- Concurrent mutations are serialized

```typescript
export const transferFunds = mutation({
  args: {
    fromId: v.id("accounts"),
    toId: v.id("accounts"),
    amount: v.number(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    const from = await ctx.db.get("accounts", args.fromId);
    const to = await ctx.db.get("accounts", args.toId);

    if (!from || !to) throw new Error("Account not found");
    if (from.balance < args.amount) throw new Error("Insufficient funds");

    // Both updates succeed or both fail
    await ctx.db.patch("accounts", args.fromId, { balance: from.balance - args.amount });
    await ctx.db.patch("accounts", args.toId, { balance: to.balance + args.amount });

    return null;
  },
});
```

## Reading Data in Mutations

Mutations can read data with the same methods as queries:

```typescript
export const sendMessage = mutation({
  args: {
    channelId: v.id("channels"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Validate channel exists
    const channel = await ctx.db.get("channels", args.channelId);
    if (!channel) throw new Error("Channel not found");

    await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
    });

    return null;
  },
});
```

## Scheduling from Mutations

```typescript
import { internal } from "./_generated/api";

export const sendMessage = mutation({
  args: { channelId: v.id("channels"), content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
    });

    // Schedule background action (runs after mutation commits)
    await ctx.scheduler.runAfter(0, internal.ai.generateResponse, {
      channelId: args.channelId,
    });

    return null;
  },
});
```

**Important:** Scheduling is transactional — if mutation fails, job isn't scheduled.

## Calling Mutations from Other Functions

```typescript
import { api, internal } from "./_generated/api";

// From mutation:
await ctx.runMutation(api.users.update, { userId, name });

// From action:
await ctx.runMutation(internal.users.updateStats, { userId });
```

**TypeScript tip:** When calling a mutation in the same file, add a type annotation:

```typescript
const id: Id<"users"> = await ctx.runMutation(api.users.create, { name: "test" });
```

**Race condition warning:** Minimize calls from mutations to other queries/mutations. Mutations are transactions, so splitting logic into multiple calls introduces race conditions. Prefer doing all reads and writes within a single mutation.

## Complete Example

```typescript
import { mutation, internalMutation } from "./_generated/server";
import { v } from "convex/values";
import { internal } from "./_generated/api";

export const sendMessage = mutation({
  args: {
    channelId: v.id("channels"),
    authorId: v.id("users"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Validate entities exist
    const channel = await ctx.db.get("channels", args.channelId);
    if (!channel) throw new Error("Channel not found");

    const user = await ctx.db.get("users", args.authorId);
    if (!user) throw new Error("User not found");

    // Insert message
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      authorId: args.authorId,
      content: args.content,
    });

    // Schedule AI response
    await ctx.scheduler.runAfter(0, internal.ai.generateResponse, {
      channelId: args.channelId,
    });

    return null;
  },
});

export const writeAgentResponse = internalMutation({
  args: {
    channelId: v.id("channels"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
      // No authorId = system/AI message
    });
    return null;
  },
});
```

## Limits

- **Execution time:** max 1 second
- **Arguments:** max 8 MiB
- **Return value:** max 8 MiB
- **Data read:** max 8 MiB from database
- **Data write:** max 8 MiB to database
- **Documents read:** max 16,384 documents
- **Documents write:** max 8,192 documents
- **Implicit return:** If no explicit return, returns `null`

**Important:** Hitting any limit causes the function call to fail with an error. Design your application to stay within these limits.

**Design tip:** If you need to store large amounts of data (e.g., stock ticker prices over time), don't create a document per data point. Instead, save data as JSON to file storage and have the client download and render it.
