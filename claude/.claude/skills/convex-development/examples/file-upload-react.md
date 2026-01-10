# Example: File Upload with React

Complete example showing image upload flow in a chat application.

## Backend (Convex)

### Schema

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  messages: defineTable({
    body: v.string(),
    author: v.string(),
    format: v.union(v.literal("text"), v.literal("image")),
  }),
});
```

### Mutations and Queries

```typescript
// convex/messages.ts
import { v } from "convex/values";
import { query, mutation } from "./_generated/server";

// List messages with image URLs resolved
export const list = query({
  args: {},
  handler: async (ctx) => {
    const messages = await ctx.db.query("messages").collect();
    return Promise.all(
      messages.map(async (message) => ({
        ...message,
        // If the message is an "image", its "body" is an Id<"_storage">
        ...(message.format === "image"
          ? { url: await ctx.storage.getUrl(message.body as any) }
          : {}),
      }))
    );
  },
});

// Generate a short-lived upload URL
export const generateUploadUrl = mutation({
  args: {},
  returns: v.string(),
  handler: async (ctx) => {
    return await ctx.storage.generateUploadUrl();
  },
});

// Save an image message to the database
export const sendImage = mutation({
  args: {
    storageId: v.id("_storage"),
    author: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      body: args.storageId,
      author: args.author,
      format: "image",
    });
    return null;
  },
});

// Save a text message to the database
export const sendMessage = mutation({
  args: {
    body: v.string(),
    author: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      body: args.body,
      author: args.author,
      format: "text",
    });
    return null;
  },
});
```

## Frontend (React)

```tsx
// src/App.tsx
import { FormEvent, useRef, useState } from "react";
import { useMutation, useQuery } from "convex/react";
import { api } from "../convex/_generated/api";

export default function App() {
  const messages = useQuery(api.messages.list) || [];

  const [newMessageText, setNewMessageText] = useState("");
  const sendMessage = useMutation(api.messages.sendMessage);

  const [name] = useState(() => "User " + Math.floor(Math.random() * 10000));

  async function handleSendMessage(event: FormEvent) {
    event.preventDefault();
    if (newMessageText) {
      await sendMessage({ body: newMessageText, author: name });
    }
    setNewMessageText("");
  }

  // Image upload state and mutations
  const generateUploadUrl = useMutation(api.messages.generateUploadUrl);
  const sendImage = useMutation(api.messages.sendImage);

  const imageInput = useRef<HTMLInputElement>(null);
  const [selectedImage, setSelectedImage] = useState<File | null>(null);

  async function handleSendImage(event: FormEvent) {
    event.preventDefault();

    // Step 1: Get a short-lived upload URL
    const postUrl = await generateUploadUrl();

    // Step 2: POST the file to the URL
    const result = await fetch(postUrl, {
      method: "POST",
      headers: { "Content-Type": selectedImage!.type },
      body: selectedImage,
    });
    const json = await result.json();
    if (!result.ok) {
      throw new Error(`Upload failed: ${JSON.stringify(json)}`);
    }
    const { storageId } = json;

    // Step 3: Save the storage ID to the database
    await sendImage({ storageId, author: name });

    // Reset form
    setSelectedImage(null);
    imageInput.current!.value = "";
  }

  return (
    <main>
      <h1>Convex Chat</h1>
      <p className="badge">
        <span>{name}</span>
      </p>

      {/* Message list */}
      <ul>
        {messages.map((message) => (
          <li key={message._id}>
            <span>{message.author}:</span>
            {message.format === "image" ? (
              <img src={message.url} height="300px" width="auto" alt="uploaded" />
            ) : (
              <span>{message.body}</span>
            )}
            <span>{new Date(message._creationTime).toLocaleTimeString()}</span>
          </li>
        ))}
      </ul>

      {/* Text message form */}
      <form onSubmit={handleSendMessage}>
        <input
          value={newMessageText}
          onChange={(event) => setNewMessageText(event.target.value)}
          placeholder="Write a message…"
        />
        <input type="submit" value="Send" disabled={!newMessageText} />
      </form>

      {/* Image upload form */}
      <form onSubmit={handleSendImage}>
        <input
          type="file"
          accept="image/*"
          ref={imageInput}
          onChange={(event) => setSelectedImage(event.currentTarget.files![0])}
          disabled={selectedImage !== null}
        />
        <input
          type="submit"
          value="Send Image"
          disabled={selectedImage === null}
        />
      </form>
    </main>
  );
}
```

## Upload Flow Summary

1. **Generate upload URL** — Call `generateUploadUrl` mutation to get a short-lived URL
2. **POST file** — Upload the file directly to the URL with appropriate Content-Type
3. **Extract storageId** — Get the `storageId` from the response JSON
4. **Save to database** — Call a mutation to store the `storageId` (not the URL!)
5. **Display** — Query messages and resolve `storageId` to URL via `ctx.storage.getUrl()`

## Key Points

- **Store IDs, not URLs** — URLs are signed and expire; storage IDs are permanent
- **Resolve URLs on read** — Use `ctx.storage.getUrl()` when returning data to client
- **Content-Type matters** — Set the correct MIME type when uploading
- **Handle errors** — Check `result.ok` before extracting storageId
