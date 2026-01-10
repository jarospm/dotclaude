# Convex Scheduling Reference

Convex provides two ways to run code in the background: crons (recurring) and the scheduler (one-off).

## Cron Jobs

Define in `convex/crons.ts`:

```typescript
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

// Run every 2 hours
crons.interval("cleanup old data", { hours: 2 }, internal.cleanup.run, {});

// Run on a cron schedule (every day at midnight UTC)
crons.cron("daily report", "0 0 * * *", internal.reports.generateDaily, {});

export default crons;
```

### Cron Methods

**`crons.interval(name, interval, functionRef, args)`**

```typescript
crons.interval("name", { hours: 2 }, internal.module.function, {});
crons.interval("name", { minutes: 30 }, internal.module.function, {});
crons.interval("name", { seconds: 60 }, internal.module.function, {});
```

**`crons.cron(name, cronExpression, functionRef, args)`**

```typescript
// Standard cron syntax: minute hour day-of-month month day-of-week
crons.cron("daily", "0 0 * * *", internal.module.function, {});      // midnight
crons.cron("hourly", "0 * * * *", internal.module.function, {});     // every hour
crons.cron("weekdays", "0 9 * * 1-5", internal.module.function, {}); // 9am weekdays
```

### Cron Rules

- **Only use `crons.interval()` or `crons.cron()`** — Don't use deprecated helpers like `crons.hourly()`
- **Pass function references, not functions** — Use `internal.module.function`, not the function itself
- **Functions can be in the same file** — Import `internal` even for same-file functions
- **Export crons as default**

### Complete Cron Example

```typescript
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";
import { internalAction } from "./_generated/server";
import { v } from "convex/values";

// Define the cron handler in the same file
export const cleanupOldMessages = internalAction({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    // Cleanup logic here
    console.log("Running cleanup...");
    return null;
  },
});

const crons = cronJobs();

crons.interval(
  "cleanup old messages",
  { hours: 24 },
  internal.crons.cleanupOldMessages,
  {}
);

export default crons;
```

## Scheduler (One-off Jobs)

Schedule jobs from mutations or actions using `ctx.scheduler`:

```typescript
import { mutation } from "./_generated/server";
import { internal } from "./_generated/api";

export const sendMessage = mutation({
  args: { channelId: v.id("channels"), content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
    });

    // Schedule immediately (0ms delay)
    await ctx.scheduler.runAfter(0, internal.ai.generateResponse, {
      channelId: args.channelId,
    });

    // Schedule with delay (5 seconds)
    await ctx.scheduler.runAfter(5000, internal.notifications.send, {
      channelId: args.channelId,
    });

    return null;
  },
});
```

### Scheduler Methods

**`ctx.scheduler.runAfter(delayMs, functionRef, args)`**

```typescript
// Run immediately
await ctx.scheduler.runAfter(0, internal.module.function, { arg: "value" });

// Run after 1 minute
await ctx.scheduler.runAfter(60000, internal.module.function, { arg: "value" });

// Run after 1 hour
await ctx.scheduler.runAfter(3600000, internal.module.function, { arg: "value" });
```

**`ctx.scheduler.runAt(timestamp, functionRef, args)`**

```typescript
// Run at specific time
const futureTime = Date.now() + 86400000; // 24 hours from now
await ctx.scheduler.runAt(futureTime, internal.module.function, { arg: "value" });
```

**Important:** You MUST pass a function reference (`api.module.fn` or `internal.module.fn`), not a string or the function itself.

```typescript
// WRONG:
await ctx.scheduler.runAfter(0, "internal.ai.generate", {});     // String - won't work!
await ctx.scheduler.runAfter(0, myFunction, {});                  // Direct function - won't work!

// CORRECT:
await ctx.scheduler.runAfter(0, internal.ai.generate, {});        // Function reference
```

## Critical: Auth Does NOT Propagate

**Scheduled jobs run with `null` auth context.**

```typescript
// BAD: This won't work - auth is null in scheduled job
export const sendMessage = mutation({
  handler: async (ctx, args) => {
    await ctx.scheduler.runAfter(0, api.messages.process, {});
    // The scheduled job won't have user auth!
  },
});

// GOOD: Use internal functions and pass user ID explicitly
export const sendMessage = mutation({
  handler: async (ctx, args) => {
    const userId = ctx.auth.getUserIdentity()?.subject;
    await ctx.scheduler.runAfter(0, internal.messages.process, {
      userId,  // Pass explicitly
    });
  },
});
```

## Scheduler is Transactional

When called from a mutation, scheduling is part of the transaction:
- If mutation fails, job is NOT scheduled
- If mutation succeeds, job is guaranteed to be scheduled

```typescript
export const createOrderWithNotification = mutation({
  handler: async (ctx, args) => {
    // If this throws, notification won't be scheduled
    const orderId = await ctx.db.insert("orders", { ... });

    // Only scheduled if insert succeeds
    await ctx.scheduler.runAfter(0, internal.notifications.orderCreated, {
      orderId,
    });

    return orderId;
  },
});
```

## Best Practices

- **Use internal functions** for scheduled jobs (they have more privileges)
- **Pass IDs, not auth** — Auth doesn't propagate; pass user/resource IDs
- **Minimum 10 second intervals** — Don't schedule tighter loops
- **Prefer scheduling actions** for external API calls
- **Keep jobs idempotent** — Jobs may occasionally run more than once
