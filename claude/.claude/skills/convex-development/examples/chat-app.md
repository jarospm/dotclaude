# Example: Real-Time Chat App with AI

A complete Convex backend for a real-time chat application with AI responses.

## Features

- User creation
- Channel-based conversations
- Real-time message updates
- Automatic AI responses using OpenAI

## Schema Design

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  channels: defineTable({
    name: v.string(),
  }),

  users: defineTable({
    name: v.string(),
  }),

  messages: defineTable({
    channelId: v.id("channels"),
    authorId: v.optional(v.id("users")),  // optional for AI messages
    content: v.string(),
  }).index("by_channel", ["channelId"]),
});
```

## API Design

**Public mutations:**
- `createUser` — Create a new user
- `createChannel` — Create a new channel
- `sendMessage` — Send a message and trigger AI response

**Public queries:**
- `listMessages` — Get recent messages from a channel

**Internal functions:**
- `generateResponse` — Generate AI response (action)
- `loadContext` — Load message context for AI (query)
- `writeAgentResponse` — Save AI response (mutation)

## Implementation

```typescript
// convex/index.ts
import {
  query,
  mutation,
  internalQuery,
  internalMutation,
  internalAction,
} from "./_generated/server";
import { v } from "convex/values";
import OpenAI from "openai";
import { internal } from "./_generated/api";

// ============ Public Mutations ============

export const createUser = mutation({
  args: { name: v.string() },
  returns: v.id("users"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("users", { name: args.name });
  },
});

export const createChannel = mutation({
  args: { name: v.string() },
  returns: v.id("channels"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("channels", { name: args.name });
  },
});

export const sendMessage = mutation({
  args: {
    channelId: v.id("channels"),
    authorId: v.id("users"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Validate channel exists
    const channel = await ctx.db.get(args.channelId);
    if (!channel) throw new Error("Channel not found");

    // Validate user exists
    const user = await ctx.db.get(args.authorId);
    if (!user) throw new Error("User not found");

    // Insert message
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      authorId: args.authorId,
      content: args.content,
    });

    // Schedule AI response
    await ctx.scheduler.runAfter(0, internal.index.generateResponse, {
      channelId: args.channelId,
    });

    return null;
  },
});

// ============ Public Queries ============

export const listMessages = query({
  args: { channelId: v.id("channels") },
  returns: v.array(
    v.object({
      _id: v.id("messages"),
      _creationTime: v.number(),
      channelId: v.id("channels"),
      authorId: v.optional(v.id("users")),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(10);
  },
});

// ============ Internal Functions ============

const openai = new OpenAI();

export const generateResponse = internalAction({
  args: { channelId: v.id("channels") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Load conversation context
    const context = await ctx.runQuery(internal.index.loadContext, {
      channelId: args.channelId,
    });

    // Generate AI response
    const response = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: context,
    });

    const content = response.choices[0].message.content;
    if (!content) throw new Error("No content in response");

    // Save response
    await ctx.runMutation(internal.index.writeAgentResponse, {
      channelId: args.channelId,
      content,
    });

    return null;
  },
});

export const loadContext = internalQuery({
  args: { channelId: v.id("channels") },
  returns: v.array(
    v.object({
      role: v.union(v.literal("user"), v.literal("assistant")),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    const channel = await ctx.db.get(args.channelId);
    if (!channel) throw new Error("Channel not found");

    const messages = await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(10);

    const result = [];
    for (const message of messages) {
      if (message.authorId) {
        const user = await ctx.db.get(message.authorId);
        if (!user) throw new Error("User not found");
        result.push({
          role: "user" as const,
          content: `${user.name}: ${message.content}`,
        });
      } else {
        result.push({
          role: "assistant" as const,
          content: message.content,
        });
      }
    }

    return result;
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
      // No authorId = AI message
    });
    return null;
  },
});
```

## Configuration Files

### package.json

```json
{
  "name": "chat-app",
  "version": "1.0.0",
  "dependencies": {
    "convex": "^1.31.2",
    "openai": "^4.79.0"
  },
  "devDependencies": {
    "typescript": "^5.7.3"
  }
}
```

### convex/tsconfig.json

```json
{
  "compilerOptions": {
    "allowJs": true,
    "strict": true,
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "target": "ESNext",
    "lib": ["ES2021", "dom"],
    "forceConsistentCasingInFileNames": true,
    "module": "ESNext",
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["./**/*"],
  "exclude": ["./_generated"]
}
```

## Key Patterns Demonstrated

1. **Schema with indexes** — `by_channel` index for efficient message queries
2. **Public vs internal functions** — API separation for security
3. **Validation** — Check entities exist before operations
4. **Scheduling** — Background AI response generation
5. **Action → Query → Mutation flow** — Actions orchestrate data access
6. **Optional fields** — `authorId` optional for system/AI messages
7. **Type safety** — `as const` for literal types in union
