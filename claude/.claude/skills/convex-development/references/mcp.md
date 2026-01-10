# Convex MCP Server Reference

The Model Context Protocol (MCP) allows AI agents to interact with Convex deployments directly. The Convex MCP server supports introspecting tables and functions, running queries/mutations/actions, reading data, and managing environment variables.

## Quick Reference

### Workflow Pattern

```
1. status(projectDir) → get deploymentSelector
2. Use deploymentSelector for all subsequent calls
```

The `deploymentSelector` is an opaque token encoding the project directory and deployment type. Always pass it unchanged to subsequent tools.

### Deployment Types

- **ownDev** — Your personal development deployment (default for most work)
- **prod** — Production deployment (use carefully)
- **urlWithAdminKey** — Local deployment when running `npx convex dev`

**Best practice:** Default to `ownDev` unless debugging production-specific issues.

## Available Tools

### status

Get available deployments for a project.

```
Args:
  projectDir: string (optional) — Root directory containing convex/ folder

Returns:
  availableDeployments: Array of {kind, deploymentSelector, url, dashboardUrl}
```

**Example use case:** Start any MCP session by calling status to get the deployment selector.

### tables

List all tables with declared and inferred schemas.

```
Args:
  deploymentSelector: string (required) — From status tool

Returns:
  tables: Object mapping table names to {schema, inferredSchema}
```

**Use cases:**
- Understand data model before writing queries
- Find unused optional fields in schema
- Suggest schema improvements for `v.any()` types
- Verify indexes exist before using `withIndex()`

### data

Read paginated data from a table.

```
Args:
  deploymentSelector: string (required)
  tableName: string (required)
  order: "asc" | "desc" (required)
  limit: number (optional) — Max results, default 100, max 1000
  cursor: string (optional) — Continue from previous page

Returns:
  page: Array of documents
  isDone: boolean
  continueCursor: string | null
```

**Best practices:**
- Use `limit` to avoid fetching too much data
- Use `order: "desc"` to see most recent documents first
- For large tables, prefer `runOneoffQuery` with filters

### functionSpec

Get metadata for all deployed functions.

```
Args:
  deploymentSelector: string (required)

Returns:
  Array of function specs with:
    - identifier (path:functionName)
    - args validator
    - returns validator
    - type (query/mutation/action)
    - visibility (public/internal)
```

**Use cases:**
- Understand available API before calling functions
- Find the right function to call
- Check argument requirements

### run

Execute a Convex function (query, mutation, or action).

```
Args:
  deploymentSelector: string (required)
  functionName: string (required) — e.g., "path/to/module:functionName"
  args: string (required) — JSON-encoded arguments object

Returns:
  result: Function return value
  logLines: Array of console output
```

**Important:**
- Use JSON.stringify for the args parameter
- Internal functions can be called (no authentication required via MCP)
- Mutations will modify the database — use carefully on prod

**Example:**
```
functionName: "topics:getByName"
args: "{\"name\": \"english idioms\"}"
```

### runOneoffQuery

Execute ad-hoc JavaScript code as a read-only query.

```
Args:
  deploymentSelector: string (required)
  query: string (required) — JavaScript code following the template below

Returns:
  result: Query return value
  logLines: Array of console output
```

**Template:**
```javascript
import { query } from "convex:/_system/repl/wrappers.js";

export default query({
  handler: async (ctx) => {
    // Your query code here
    // Can use: ctx.db.query(), ctx.db.get(), ctx.storage.getUrl()
    // CANNOT: write to db, call external APIs
    return result;
  },
});
```

**Why this is powerful:**
- Fully sandboxed — cannot modify data
- No imports available except the wrapper
- Great for ad-hoc analysis, aggregations, joins
- More flexible than the `data` tool for complex queries

**Example — Count by status:**
```javascript
import { query } from "convex:/_system/repl/wrappers.js";

export default query({
  handler: async (ctx) => {
    const items = await ctx.db.query("contentQueue").collect();
    return {
      pending: items.filter(i => i.status === "pending").length,
      sent: items.filter(i => i.status === "sent").length,
      failed: items.filter(i => i.status === "failed").length,
    };
  },
});
```

### logs

Fetch recent log entries from the deployment.

```
Args:
  deploymentSelector: string (required)
  cursor: number (optional) — Timestamp in ms to start from (0 for beginning)
  entriesLimit: number (optional) — Max entries, max 1000
  tokensLimit: number (optional) — Approx max tokens, default 20000
  jsonl: boolean (optional) — Return raw JSONL instead of formatted text

Returns:
  Log entries and new cursor for pagination
```

**Use cases:**
- Debug function execution
- Monitor errors
- Trace function calls

### Environment Variables

#### envList
```
Args: deploymentSelector
Returns: Array of environment variable names
```

#### envGet
```
Args: deploymentSelector, name
Returns: Environment variable value
```

#### envSet
```
Args: deploymentSelector, name, value
Returns: Success confirmation
```

#### envRemove
```
Args: deploymentSelector, name
Returns: Success confirmation
```

**Caution:** Environment variable changes take effect immediately. Be careful on production deployments.

## Best Practices

### 1. Always Start with Status

Every MCP session should begin with a `status` call to get the deployment selector:

```
status(projectDir: "/path/to/project")
→ Extract deploymentSelector from response
→ Use for all subsequent calls
```

### 2. Prefer Development Deployments

Default to `ownDev` for exploration and testing. Only use `prod` when:
- Debugging production-specific issues
- Verifying production data state
- Making intentional production changes

### 3. Use runOneoffQuery for Complex Analysis

The `data` tool is great for browsing, but `runOneoffQuery` is more powerful for:
- Filtering data (no full table scans sent over the wire)
- Joining across tables
- Aggregations and counts
- Complex business logic

### 4. Check Tables Before Querying

Always call `tables` first to understand:
- Available table names
- Field types and optionality
- Available indexes (use `withIndex()` when possible)

### 5. Check functionSpec Before run

Before calling a function with `run`:
- Verify it exists and is the right type
- Check required arguments
- Understand return type

### 6. Be Careful with Mutations

The `run` tool can execute mutations that modify data. On production:
- Double-check the deployment selector
- Verify arguments before execution
- Consider testing on dev first

## Common Patterns

### Explore a Deployment

```
1. status → get selector
2. tables → understand schema
3. functionSpec → see available APIs
4. data or runOneoffQuery → inspect data
```

### Debug an Issue

```
1. status → get selector
2. logs → check recent errors
3. data/runOneoffQuery → inspect related data
4. run → test specific functions
```

### Data Analysis

```
1. status → get selector
2. tables → find relevant tables
3. runOneoffQuery → write custom aggregation/analysis
```

## Limitations

- **No imports in runOneoffQuery** — Only the built-in wrapper is available
- **Read-only queries** — runOneoffQuery cannot write to the database
- **No streaming** — Results are returned in full, not streamed
- **String args only** — All tool arguments should be strings (JSON-encode objects)
