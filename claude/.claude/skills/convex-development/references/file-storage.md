# Convex File Storage Reference

Convex includes built-in file storage for images, videos, PDFs, and other files.

## Upload Flow

1. Generate upload URL (mutation)
2. Upload file to URL (client)
3. Store the returned file ID (mutation)

### Generate Upload URL

```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const generateUploadUrl = mutation({
  args: {},
  returns: v.string(),
  handler: async (ctx) => {
    return await ctx.storage.generateUploadUrl();
  },
});
```

### Client Upload (React Example)

```typescript
const generateUploadUrl = useMutation(api.files.generateUploadUrl);
const saveFile = useMutation(api.files.saveFile);

async function handleUpload(file: File) {
  // 1. Get upload URL
  const uploadUrl = await generateUploadUrl();

  // 2. Upload file
  const response = await fetch(uploadUrl, {
    method: "POST",
    headers: { "Content-Type": file.type },
    body: file,
  });
  const { storageId } = await response.json();

  // 3. Save file ID to database
  await saveFile({ storageId, fileName: file.name });
}
```

### Save File Reference

```typescript
export const saveFile = mutation({
  args: {
    storageId: v.id("_storage"),
    fileName: v.string(),
  },
  returns: v.id("files"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("files", {
      storageId: args.storageId,
      fileName: args.fileName,
    });
  },
});
```

## Retrieve Files

### Get Signed URL

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const getFileUrl = query({
  args: { storageId: v.id("_storage") },
  returns: v.union(v.string(), v.null()),
  handler: async (ctx, args) => {
    return await ctx.storage.getUrl(args.storageId);
  },
});
```

**Note:** `getUrl()` returns `null` if the file doesn't exist.

### Get File Metadata

Query the `_storage` system table:

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

type FileMetadata = {
  _id: Id<"_storage">;
  _creationTime: number;
  contentType?: string;
  sha256: string;
  size: number;
};

export const getFileMetadata = query({
  args: { storageId: v.id("_storage") },
  returns: v.union(
    v.object({
      _id: v.id("_storage"),
      _creationTime: v.number(),
      contentType: v.optional(v.string()),
      sha256: v.string(),
      size: v.number(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db.system.get(args.storageId);
  },
});
```

**Don't use deprecated `ctx.storage.getMetadata()`.**

## Delete Files

```typescript
export const deleteFile = mutation({
  args: { storageId: v.id("_storage") },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.storage.delete(args.storageId);
    return null;
  },
});
```

## Store File IDs, Not URLs

**Wrong:**
```typescript
// Don't store URLs - they expire!
await ctx.db.insert("users", {
  name: "Alice",
  avatarUrl: "https://convex.cloud/...",  // BAD
});
```

**Correct:**
```typescript
// Store file IDs
await ctx.db.insert("users", {
  name: "Alice",
  avatarId: storageId,  // GOOD
});

// Generate URL when needed
const avatarUrl = await ctx.storage.getUrl(user.avatarId);
```

## Schema with File References

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    avatarId: v.optional(v.id("_storage")),
  }),

  documents: defineTable({
    title: v.string(),
    fileId: v.id("_storage"),
    uploadedBy: v.id("users"),
  }),
});
```

## Complete Example: Image Upload in Chat

```typescript
// convex/schema.ts
export default defineSchema({
  messages: defineTable({
    channelId: v.id("channels"),
    content: v.optional(v.string()),
    imageId: v.optional(v.id("_storage")),
  }).index("by_channel", ["channelId"]),
});

// convex/messages.ts
export const generateUploadUrl = mutation({
  args: {},
  returns: v.string(),
  handler: async (ctx) => {
    return await ctx.storage.generateUploadUrl();
  },
});

export const sendImageMessage = mutation({
  args: {
    channelId: v.id("channels"),
    imageId: v.id("_storage"),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      imageId: args.imageId,
    });
    return null;
  },
});

export const getMessages = query({
  args: { channelId: v.id("channels") },
  handler: async (ctx, args) => {
    const messages = await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .collect();

    // Add image URLs to messages
    return Promise.all(
      messages.map(async (msg) => ({
        ...msg,
        imageUrl: msg.imageId ? await ctx.storage.getUrl(msg.imageId) : null,
      }))
    );
  },
});
```

## Working with Blobs

Convex storage works with `Blob` objects. Convert as needed:

```typescript
// String to Blob
const blob = new Blob(["Hello World"], { type: "text/plain" });

// ArrayBuffer to Blob
const blob = new Blob([arrayBuffer], { type: "application/octet-stream" });

// JSON to Blob
const blob = new Blob([JSON.stringify(data)], { type: "application/json" });
```

## Limits

- Max file size: 20 MiB per file
- Files are stored securely and served via signed URLs
