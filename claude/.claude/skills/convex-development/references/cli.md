# Convex CLI Reference

The Convex command-line interface manages projects, deployments, and functions.

## Quick Reference

```bash
# Day-to-day operations (most used)
npx convex data                    # List tables
npx convex data myTable            # Browse table data
npx convex function-spec           # List all functions with args/returns
npx convex run myModule:func '{}'  # Run a function

# Development
npx convex dev                     # Watch mode (assume always running)
npx convex dev --once              # One-time push for verification
npx convex deploy                  # Deploy to production

# Debugging
npx convex logs                    # Tail dev logs
npx convex logs --prod             # Tail production logs

# Environment & data management
npx convex env list                # List environment variables
npx convex export --path backup.zip
npx convex import backup.zip
```

## Browsing Data

### npx convex data

Browse table data from the command line. The most common way to quickly check what's in your database.

```bash
npx convex data                       # List all tables
npx convex data users                 # Show users table
npx convex data users --limit 5       # Limit results
npx convex data users --order desc    # Most recent first
npx convex data _storage              # View system tables too
npx convex data contentQueue --prod   # View production data
```

**Typical usage:**

```bash
# Check queue status
npx convex data contentQueue --limit 10 --order desc

# See what records exist
npx convex data topics
```

## Browsing Functions

### npx convex function-spec

List all deployed functions with their argument and return type validators. Essential for understanding what APIs are available.

```bash
npx convex function-spec              # Print to stdout
npx convex function-spec --file       # Write to function-spec.json
npx convex function-spec --prod       # Inspect production deployment
```

**Output format (JSON):**

```json
{
  "url": "https://your-deployment.convex.cloud",
  "functions": [
    {
      "identifier": "messages:send",
      "functionType": "Mutation",
      "visibility": { "kind": "public" },
      "args": {
        "type": "object",
        "value": {
          "body": { "fieldType": { "type": "string" }, "optional": false }
        }
      },
      "returns": { "type": "null" }
    }
  ]
}
```

**Filtering with jq:**

```bash
# List all public mutations
npx convex function-spec | jq '.functions[] | select(.functionType == "Mutation" and .visibility.kind == "public") | .identifier'

# Find functions that take a specific arg
npx convex function-spec | jq '.functions[] | select(.args.value.userId?) | .identifier'

# List all actions
npx convex function-spec | jq '.functions[] | select(.functionType == "Action") | .identifier'
```

**Use cases:**
- Discover available APIs before calling them
- Verify what's deployed vs. what you expect
- Generate documentation or typed clients
- Debug argument mismatches

## Running Functions

### npx convex run

Execute any function (query, mutation, action) from the command line. The primary way to test and invoke functions.

```bash
# Basic usage
npx convex run messages:send '{"body": "hello", "author": "me"}'

# Run on production
npx convex run messages:list --prod

# Watch a query (re-run when data changes)
npx convex run messages:list --watch

# Push code before running
npx convex run messages:send '{"body": "test"}' --push

# Run with mock identity
npx convex run users:getMe --identity '{"name": "Test User"}'
```

**Function identifier formats:**

- `functionName` — Function exported from `convex/functionName.ts`
- `dir/file:functionName` — Function from nested path (e.g., `jobs/delivery:run`)
- `api.messages.send` — Using the generated API path

**Common patterns:**

```bash
# Test a query
npx convex run topics:getByName '{"name": "my topic"}'

# Trigger an action
npx convex run jobs/delivery:run '{"frequencies": [1, 2]}'

# Run internal function (works from CLI)
npx convex run queue:getNextPending '{}'
```

**Best practice:** Use `--push` when testing changes to ensure you're running the latest code.

## Development Workflow

### npx convex dev

The primary development command. Watches for changes and pushes to your dev deployment.

```bash
npx convex dev                        # Standard watch mode
npx convex dev --once                 # Single push, exit on completion
npx convex dev --until-success        # Retry until push succeeds
npx convex dev --run init:seed        # Run function after each push
npx convex dev --tail-logs always     # Always show logs (even during deploy)
npx convex dev --tail-logs disable    # Hide logs entirely
```

**Key options:**

- `--once` — Push once and exit. Use for CI verification or quick checks.
- `--until-success` — Keep trying until push succeeds. Useful when fixing errors.
- `--run <function>` — Run a function after each successful push (e.g., seed data).
- `--tail-logs <mode>` — Control log display: `always`, `pause-on-deploy` (default), `disable`.
- `--typecheck <mode>` — TypeScript checking: `enable`, `try` (default), `disable`.

**Best practice:** Assume `npx convex dev` is always running in a separate terminal. Use `--once` for quick verification.

### npx convex deploy

Deploy to production (or preview) deployments.

```bash
npx convex deploy                     # Deploy to production
npx convex deploy --dry-run           # Preview without deploying
npx convex deploy -y                  # Skip confirmation prompt
npx convex deploy --cmd "npm run build" --cmd-url-env-var-name VITE_CONVEX_URL
```

**Deployment target resolution:**

1. If `CONVEX_DEPLOY_KEY` is set → deploy to that key's deployment
2. If `CONVEX_DEPLOYMENT` is set → deploy to its project's production

**For CI/CD with frontend build:**

```bash
npx convex deploy --cmd "npm run build" --cmd-url-env-var-name VITE_CONVEX_URL
```

**Preview deployments:**

```bash
# With a Preview Deploy Key in CONVEX_DEPLOY_KEY:
npx convex deploy --preview-create my-feature-branch
npx convex deploy --preview-run init:seedData
```

## Debugging

### npx convex logs

Stream function execution logs.

```bash
npx convex logs                       # Tail dev logs
npx convex logs --prod                # Tail production logs
npx convex logs --history 50          # Show last 50 log entries
npx convex logs --success             # Include successful executions
npx convex logs --jsonl               # Raw JSON output
```

**Tip:** During development, `npx convex dev` already shows logs. Use `npx convex logs` when you need a separate log stream or want to see production logs.

### npx convex dashboard

Open the Convex dashboard in your browser.

```bash
npx convex dashboard    # or: npx convex dash
```

## Environment Variables

### npx convex env

Manage deployment environment variables.

```bash
# List all
npx convex env list
npx convex env list --prod

# Get/set/remove
npx convex env get API_KEY
npx convex env set API_KEY "secret-value"
npx convex env remove API_KEY

# Production
npx convex env set API_KEY "secret-value" --prod
```

**Important:** Changes take effect immediately. Be careful on production.

## Data Import/Export

### npx convex export

Export deployment data to a ZIP file.

```bash
npx convex export --path backup.zip
npx convex export --path backup.zip --include-file-storage
npx convex export --path backup.zip --prod
```

**Best practice:** Always use `--include-file-storage` for complete backups.

### npx convex import

Import data into a deployment.

```bash
# Import full snapshot
npx convex import backup.zip

# Import single table
npx convex import --table users users.jsonl

# Import modes
npx convex import backup.zip --append     # Add to existing data
npx convex import backup.zip --replace    # Replace imported tables only
npx convex import backup.zip --replace-all # Replace entire database

# Skip confirmation
npx convex import backup.zip --replace -y
```

**Supported formats:**

- **ZIP** — Snapshot export format (directory per table with `documents.jsonl`)
- **JSONLines** — One JSON object per line
- **JSON Array** — Array of objects
- **CSV** — Header row required, values parsed as numbers or strings

**Caution:** `--replace-all` deletes tables not in the import. Use carefully.

## Code Generation

### npx convex codegen

Regenerate types in `convex/_generated/`.

```bash
npx convex codegen
```

Normally handled automatically by `npx convex dev`. Useful for:
- CI verification that generated code is committed
- Recovering from corrupted generated files
- Manual regeneration after schema changes

## Common Patterns

### Development Session

```bash
# Terminal 1: Keep dev server running
npx convex dev

# Terminal 2: Work normally, use other commands as needed
npx convex function-spec | jq '.functions[] | .identifier' | head -20
npx convex data contentQueue --limit 5
npx convex run myModule:testFunction '{"arg": "value"}'
```

### Quick Verification

```bash
# Verify code compiles and pushes successfully
npx convex dev --once
```

### Database Backup/Restore

```bash
# Backup production
npx convex export --path prod-backup-$(date +%Y%m%d).zip --include-file-storage --prod

# Restore to dev for debugging
npx convex import prod-backup-20241221.zip --replace
```

### Debugging Production Issues

```bash
# Stream production logs
npx convex logs --prod

# Check specific data
npx convex data contentQueue --prod --limit 10 --order desc

# Check what functions are deployed
npx convex function-spec --prod | jq '.functions | length'

# Run diagnostic query
npx convex run diagnostics:checkQueue --prod
```

## Targeting Deployments

Most commands support these targeting options:

- `--prod` — Target production deployment
- `--preview-name <name>` — Target a preview deployment
- `--deployment-name <name>` — Target a specific deployment by name
- `--env-file <path>` — Use custom environment file

**Default behavior:** Commands target your dev deployment (from `CONVEX_DEPLOYMENT` in `.env.local`).

## CI/CD Integration

### GitHub Actions Example

```yaml
- name: Deploy to Convex
  env:
    CONVEX_DEPLOY_KEY: ${{ secrets.CONVEX_DEPLOY_KEY }}
  run: |
    npx convex deploy --cmd "npm run build" --cmd-url-env-var-name VITE_CONVEX_URL
```

### Vercel/Netlify

Set `CONVEX_DEPLOY_KEY` in the hosting platform's environment variables. Use a Preview Deploy Key for preview deployments.
