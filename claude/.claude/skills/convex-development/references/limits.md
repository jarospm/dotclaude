# Convex Limits Reference

Convex enforces limits to ensure fast, predictable performance. Hitting any limit causes the function call to fail with an error.

## Function Limits

### Execution Time

- **Queries:** max 1 second
- **Mutations:** max 1 second
- **Actions:** max 10 minutes
- **HTTP actions:** max 10 minutes

### Arguments and Return Values

- **Arguments:** max 8 MiB
- **Return value:** max 8 MiB

## Database Limits

### Read Limits

- **Data read:** max 8 MiB per query/mutation
- **Documents read:** max 16,384 per query/mutation

### Write Limits

- **Data write:** max 8 MiB per mutation
- **Documents write:** max 8,192 per mutation

### Document Limits

- **Document size:** max 1 MiB per record

## Data Structure Limits

### Arrays

- **Elements:** max 8,192 per array

### Objects

- **Fields:** max 1,024 per object
- **Field names:** ASCII-only, non-empty, no `$` or `_` prefix
- **Nesting depth:** max 16 levels

### Records

- **Keys:** ASCII-only, non-empty, no `$` or `_` prefix

### Strings

- **Size:** max 1 MiB (UTF-8 encoded)

### Bytes

- **Size:** max 1 MiB

## HTTP Limits

- **Request body:** no limit
- **Streaming output:** max 20 MiB

## File Storage Limits

- **File size:** max 20 MiB per file

## Design Strategies for Limits

### High-Volume Data

If you need to store large amounts of data (e.g., stock prices over time):

```typescript
// DON'T: Create a document per data point
await ctx.db.insert("prices", { symbol: "AAPL", price: 150.25, time: Date.now() });
// This will hit document limits quickly!

// DO: Save as JSON to file storage
const blob = new Blob([JSON.stringify(priceHistory)], { type: "application/json" });
const storageId = await ctx.storage.store(blob);
// Client downloads and renders the JSON
```

### Large Result Sets

If you need to return many documents:

```typescript
// DON'T: Return all documents at once
const all = await ctx.db.query("items").collect();  // May hit limits!

// DO: Use pagination
const page = await ctx.db
  .query("items")
  .paginate(paginationOpts);
```

### Deeply Nested Data

If you hit nesting depth limits:

```typescript
// DON'T: Deeply nested objects
const data = { a: { b: { c: { d: { e: { f: { g: { h: { i: { j: {} } } } } } } } } } };

// DO: Flatten structure or split across documents
const data = { level1: "a", level2: "b", metadata: JSON.stringify(deepData) };
```

## Quick Reference Table

| Resource | Limit |
|----------|-------|
| Query/mutation execution | 1 second |
| Action execution | 10 minutes |
| Function arguments | 8 MiB |
| Function return value | 8 MiB |
| Documents read | 16,384 |
| Documents written | 8,192 |
| Data read | 8 MiB |
| Data written | 8 MiB |
| Document size | 1 MiB |
| Array elements | 8,192 |
| Object fields | 1,024 |
| Nesting depth | 16 |
| String size | 1 MiB |
| HTTP streaming | 20 MiB |
| File size | 20 MiB |
