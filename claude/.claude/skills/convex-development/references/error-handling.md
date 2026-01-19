# Convex Error Handling Reference

Pragmatic guidelines for error handling in Convex applications.

## Two Approaches

Convex offers two idiomatic patterns for handling expected failures:

- **Return values** — Discriminated unions for internal functions, complex workflows
- **Throw errors** — `ConvexError` for public API boundaries, transaction rollback

Both are valid. Choose based on context.

## Pattern 1: Discriminated Union Returns

Return a union type that explicitly represents success or failure:

```typescript
type CreateResult =
  | { ok: true; id: Id<"users"> }
  | { ok: false; error: "EMAIL_IN_USE" | "INVALID_INPUT" };

export const create = internalMutation({
  args: { email: v.string() },
  returns: v.union(
    v.object({ ok: v.literal(true), id: v.id("users") }),
    v.object({ ok: v.literal(false), error: v.string() })
  ),
  handler: async (ctx, args): Promise<CreateResult> => {
    const existing = await ctx.db
      .query("users")
      .withIndex("by_email", (q) => q.eq("email", args.email))
      .unique();

    if (existing) {
      return { ok: false, error: "EMAIL_IN_USE" };
    }

    const id = await ctx.db.insert("users", { email: args.email });
    return { ok: true, id };
  },
});
```

### When to Use Return Values

- **Internal functions** — Callers are other Convex functions, not clients
- **Partial success is acceptable** — Caller may want to continue despite some failures
- **Aggregating results** — Processing multiple items, collecting successes and failures
- **No transaction rollback needed** — Writes before the failure should persist

### Advantages

- TypeScript enforces handling all cases
- No exception overhead
- Caller decides how to handle failures
- Easy to aggregate in loops

## Pattern 2: ConvexError Throws

Throw a `ConvexError` with structured data:

```typescript
import { ConvexError } from "convex/values";

export const assignRole = mutation({
  args: { userId: v.id("users"), role: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    const user = await ctx.db.get("users", args.userId);
    if (!user) {
      throw new ConvexError({ code: "NOT_FOUND", message: "User not found" });
    }

    if (user.role === args.role) {
      throw new ConvexError({
        code: "ALREADY_ASSIGNED",
        message: "Role already assigned",
      });
    }

    await ctx.db.patch("users", args.userId, { role: args.role });
    return null;
  },
});
```

### When to Use ConvexError

- **Public functions** — Clients need structured error information
- **Transaction rollback required** — All writes should be undone on failure
- **Deep call stacks** — Error bubbles up automatically via `runQuery`/`runMutation`/`runAction`
- **Mutations with invariants** — Failure means the operation is invalid

### Advantages

- Automatic transaction rollback in mutations
- Bubbles through `ctx.runQuery`/`runMutation`/`runAction`
- Client receives structured `error.data` payload
- Simpler than propagating return values through deep stacks

### Error Data Payloads

```typescript
// Simple string
throw new ConvexError("Role is already taken");
// error.data === "Role is already taken"

// Structured object (recommended)
throw new ConvexError({
  code: "ROLE_TAKEN",
  message: "Role is already taken",
  details: { role: "admin", takenBy: userId },
});
// error.data === { code: "ROLE_TAKEN", message: "...", details: {...} }
```

## Decision Matrix

- **Internal query/mutation** → Return value
- **Internal action calling external API** → Return value + log errors
- **Public mutation with invariants** → `ConvexError`
- **Public query** → `ConvexError` (caught by error boundary)
- **Job processing multiple items** → Return value (aggregate results)
- **Nested calls needing rollback** → `ConvexError` (bubbles automatically)

## Error Categories

### 1. Application Errors (Expected)

Logical conditions that prevent completion:

- User already exists
- Resource not found
- Invalid state transition

**Handle with:** Return values or `ConvexError`

### 2. External Service Errors

Third-party API failures:

- Rate limits
- Network timeouts
- Invalid responses

**Handle with:** Try/catch → log → return null/failure result

```typescript
export const callExternalApi = internalAction({
  args: { resourceId: v.id("resources") },
  returns: v.union(v.id("results"), v.null()),
  handler: async (ctx, args) => {
    try {
      const result = await externalApi.fetch(args.resourceId);
      const id = await ctx.runMutation(internal.results.save, { data: result });
      return id;
    } catch (error) {
      // LOG before returning null — preserves debuggability
      console.error(`External API failed for ${args.resourceId}:`, error);
      return null;
    }
  },
});
```

### 3. Developer Errors (Bugs)

Should never happen in correct code:

- Null reference
- Invalid arguments
- Logic errors

**Handle with:** Let them throw (shows in logs, caught by Convex)

### 4. Infrastructure Errors

Convex handles automatically:

- Network blips
- Internal retries

**Handle with:** Nothing — Convex retries automatically

## Job Processing Pattern

For jobs that process multiple items, use return values to continue on failure:

```typescript
type ProcessResult =
  | { status: "success"; id: Id<"results"> }
  | { status: "failed"; reason: string };

export const processItems = internalAction({
  args: { itemIds: v.array(v.id("items")) },
  returns: v.object({
    succeeded: v.number(),
    failed: v.number(),
  }),
  handler: async (ctx, args) => {
    let succeeded = 0;
    let failed = 0;

    for (const itemId of args.itemIds) {
      const result = await processItem(ctx, itemId);
      if (result.status === "success") {
        succeeded++;
      } else {
        console.error(`Failed: ${result.reason}`);
        failed++;
      }
    }

    return { succeeded, failed };
  },
});
```

## Anti-Patterns to Avoid

### 1. Silent Swallowing

```typescript
// BAD: Error disappears without trace
catch {
  return null;
}

// GOOD: Log before returning
catch (error) {
  console.error("Operation failed:", error);
  return null;
}
```

### 2. Generic Error Messages

```typescript
// BAD: No context for debugging
return { status: "failed", reason: "Operation failed" };

// GOOD: Include actionable details
return {
  status: "failed",
  reason: `Processing failed for item ${itemId}: ${error.message}`,
};
```

### 3. Throwing from Jobs

```typescript
// BAD: One failure stops entire job
for (const item of items) {
  await processItem(item); // throws on failure
}

// GOOD: Collect failures, continue processing
for (const item of items) {
  const result = await processItem(item);
  if (!result.ok) errors.push({ item, error: result.error });
}
```

### 4. Inconsistent Patterns

```typescript
// BAD: Some return null, some throw — unpredictable
export const create = /* returns null on error */
export const update = /* throws on error */

// GOOD: Consistent within a layer
// All internal actions: return value + log
// All public mutations: throw ConvexError
```

## Dev vs Production Behavior

- **Development:** Server errors show full message + stack trace
- **Production:** Server errors show generic "Server Error"
- **Both:** Application errors (`ConvexError`) show full `error.data`

**Implication:** Always use `ConvexError` for errors you want clients to see in production. Regular `throw new Error()` gets redacted.

## Client-Side Handling

```typescript
import { ConvexError } from "convex/values";

try {
  await mutation({ ... });
} catch (error) {
  if (error instanceof ConvexError) {
    // Structured error data available
    const { code, message } = error.data as { code: string; message: string };
    if (code === "NOT_FOUND") {
      // Handle specific error
    }
  } else {
    // Generic server error
  }
}
```

## Quick Reference

```typescript
import { ConvexError } from "convex/values";

// Return value pattern (internal functions)
type Result<T> = { ok: true; value: T } | { ok: false; error: string };

// ConvexError pattern (public functions)
throw new ConvexError({ code: "NOT_FOUND", message: "User not found" });

// Checking on client
if (error instanceof ConvexError) {
  const { code, message } = error.data as { code: string; message: string };
}

// Logging before returning null (actions with external APIs)
catch (error) {
  console.error(`${functionName} failed:`, error);
  return null;
}
```

## Summary

1. **Internal functions:** Return discriminated unions
2. **Public functions:** Throw `ConvexError` with structured data
3. **External API calls:** Try/catch → log → return null
4. **Jobs:** Aggregate results, don't throw
5. **Always log** before converting errors to null
6. **Be consistent** within each layer
