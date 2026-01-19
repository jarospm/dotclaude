# Convex Queries Reference

Queries read data from the database. They are automatically reactive.

## Basic Query

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const getUser = query({
  args: { userId: v.id("users") },
  returns: v.union(
    v.object({
      _id: v.id("users"),
      _creationTime: v.number(),
      name: v.string(),
      email: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db.get("users", args.userId);
  },
});
```

## Internal Query

```typescript
import { internalQuery } from "./_generated/server";

export const getPrivateData = internalQuery({
  args: { userId: v.id("users") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Only callable from other Convex functions
    return null;
  },
});
```

## Database Query Methods

### Get by ID

```typescript
const user = await ctx.db.get("users", userId);  // returns doc or null
```

### Query with Index

**Always use indexes instead of `filter()`:**

```typescript
// Good - uses index
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .collect();

// Bad - scans entire table
const messages = await ctx.db
  .query("messages")
  .filter((q) => q.eq(q.field("channelId"), channelId))  // DON'T DO THIS
  .collect();
```

### Index Query Operators

```typescript
.withIndex("index_name", (q) =>
  q.eq("field", value)           // equals
   .gt("field", value)           // greater than
   .gte("field", value)          // greater than or equal
   .lt("field", value)           // less than
   .lte("field", value)          // less than or equal
)
```

## Ordering

Default order is ascending by `_creationTime`.

```typescript
// Descending order (newest first)
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .order("desc")
  .take(10);

// Ascending order (oldest first) - default
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .order("asc")
  .take(10);
```

## Result Collection

### collect() - Get All Results

```typescript
const allMessages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .collect();
```

### take(n) - Get First N Results

```typescript
const recentMessages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .order("desc")
  .take(10);
```

### first() - Get First Result

```typescript
const latestMessage = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", channelId))
  .order("desc")
  .first();  // returns doc or null
```

### unique() - Get Exactly One Result

```typescript
const user = await ctx.db
  .query("users")
  .withIndex("by_email", (q) => q.eq("email", email))
  .unique();  // throws if 0 or >1 results
```

### Async Iteration

```typescript
for await (const message of ctx.db.query("messages")) {
  console.log(message.content);
  // Don't use .collect() or .take() with async iteration
}
```

## Pagination

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";
import { paginationOptsValidator } from "convex/server";

export const listMessages = query({
  args: {
    channelId: v.id("channels"),
    paginationOpts: paginationOptsValidator,
  },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
```

**Pagination options:**
- `numItems: number` — Max documents to return
- `cursor: string | null` — Cursor for next page (null for first page)

**Pagination result:**
- `page: Doc[]` — Array of documents
- `isDone: boolean` — Whether this is the last page
- `continueCursor: string` — Cursor for next page

## Full-Text Search

```typescript
const results = await ctx.db
  .query("messages")
  .withSearchIndex("search_content", (q) =>
    q.search("content", "hello world")
     .eq("channelId", channelId)  // optional filter
  )
  .take(10);
```

## Calling Queries from Other Functions

```typescript
import { api, internal } from "./_generated/api";

// From mutation or action:
const user = await ctx.runQuery(api.users.get, { userId });
const data = await ctx.runQuery(internal.users.getPrivate, { userId });
```

**TypeScript tip:** When calling a query in the same file, add a type annotation:

```typescript
const result: string = await ctx.runQuery(api.example.myQuery, { name: "test" });
```

**Race condition warning:** Minimize calls from actions to queries. Queries are transactions, so splitting logic into multiple calls introduces race conditions. Prefer fewer, larger queries over many small ones.

## Limits

- **Execution time:** max 1 second
- **Arguments:** max 8 MiB
- **Return value:** max 8 MiB
- **Data read:** max 8 MiB from database
- **Documents read:** max 16,384 documents
- **Implicit return:** If no explicit return, returns `null`

**Important:** Hitting any limit causes the function call to fail with an error. Design your application to stay within these limits.
