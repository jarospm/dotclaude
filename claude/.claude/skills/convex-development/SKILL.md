---
name: Convex Development
description: This skill MUST be used when building with Convex. Use it when writing queries, mutations, actions, schemas, validators, HTTP endpoints, or any Convex backend code. Covers function syntax, database operations, scheduling, file storage, and TypeScript patterns.
version: 1.0.0
---

# Convex Development Skill

This is the authoritative guide for building Convex applications. **Always use this skill when working with Convex.**

## Critical: New Function Syntax

ALWAYS use the new function syntax for ALL Convex functions:

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const myFunction = query({
  args: { name: v.string() },
  returns: v.string(),
  handler: async (ctx, args) => {
    return `Hello ${args.name}`;
  },
});
```

**Requirements:**
- ALWAYS include `args` validator (use `args: {}` if no arguments)
- ALWAYS include `returns` validator (use `returns: v.null()` if no return value)
- Handler receives `ctx` and `args` parameters

## Function Types

**Public functions** (exposed to clients):
- `query` — Read-only database access
- `mutation` — Read/write database access
- `action` — External API calls, no direct db access

**Internal functions** (only callable from other Convex functions):
- `internalQuery`, `internalMutation`, `internalAction`

All imported from `./_generated/server`.

## Function References

Use `api` for public functions, `internal` for internal functions:

```typescript
import { api, internal } from "./_generated/api";

// Call from mutation/action:
await ctx.runQuery(api.users.get, { id: userId });
await ctx.runMutation(internal.users.updateStats, { id: userId });
```

File-based routing: `convex/users.ts` → `api.users.functionName`

## Key Rules

1. **No `filter()` in queries** — Use `withIndex()` instead
2. **Actions can't access `ctx.db`** — Call mutations/queries instead
3. **Scheduled jobs lose auth** — Use internal functions for privileged ops
4. **Store file IDs, not URLs** — URLs expire, IDs don't

## Reference Files

Consult these for detailed patterns:

### Core References
- **`references/validators.md`** — All validator types and usage patterns
- **`references/schema.md`** — Schema design, indexes, search indexes
- **`references/typescript.md`** — TypeScript types, Id<>, Doc<>, strict typing

### Function References
- **`references/queries.md`** — Query patterns, pagination, ordering, full-text search
- **`references/mutations.md`** — Database operations, transactions, ctx.db methods
- **`references/actions.md`** — Actions, HTTP endpoints, external APIs

### Advanced References
- **`references/scheduling.md`** — Crons, ctx.scheduler, background jobs
- **`references/file-storage.md`** — File uploads, signed URLs, metadata
- **`references/error-handling.md`** — ConvexError, return values, error patterns
- **`references/limits.md`** — All Convex limits and design strategies

### Tools & Operations
- **`references/cli.md`** — CLI commands for data, functions, logs, deploy
- **`references/mcp.md`** — MCP server tools for AI agent access

### External Documentation
- **`references/docs-index.md`** — Index of all Convex docs for WebFetch fallback

### Examples
- **`examples/chat-app.md`** — Complete real-time chat with AI responses
- **`examples/file-upload-react.md`** — File upload flow with React frontend

## Quick Validator Reference

```typescript
v.string()           // string
v.number()           // number (float64)
v.boolean()          // boolean
v.null()             // null (NOT undefined)
v.int64()            // bigint
v.bytes()            // ArrayBuffer
v.id("tableName")    // document ID
v.array(v.string())  // array
v.object({ ... })    // object with known fields
v.record(k, v)       // object with dynamic keys
v.optional(v.X())    // optional field
v.union(v.X(), v.Y())// union type
v.literal("value")   // literal string
```

## When to Read Reference Files

- **Writing a query?** → Read `references/queries.md`
- **Writing a mutation?** → Read `references/mutations.md`
- **Calling external APIs?** → Read `references/actions.md`
- **Designing tables?** → Read `references/schema.md`
- **Complex validators?** → Read `references/validators.md`
- **Background jobs?** → Read `references/scheduling.md`
- **File uploads?** → Read `references/file-storage.md`
- **Type issues?** → Read `references/typescript.md`
- **Error handling?** → Read `references/error-handling.md`
- **CLI commands?** → Read `references/cli.md`
- **Using MCP tools?** → Read `references/mcp.md`
- **Hitting limits?** → Read `references/limits.md`

## When Local References Don't Cover Your Use Case

If the local reference files don't cover a topic (e.g., authentication, vector search, React Native, testing):

1. Read `references/docs-index.md` to find the relevant topic
2. WebFetch the page from `https://docs.convex.dev{path}`

**Example:** For Clerk authentication, find `/auth/clerk.md` in the index, then fetch `https://docs.convex.dev/auth/clerk`
