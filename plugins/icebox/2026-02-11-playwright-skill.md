# playwright-skill

**Source:** https://github.com/lackeyjb/playwright-skill
**Type:** Claude Code skill/plugin

## What it does

Code-generation approach to browser automation. Instead of calling
pre-built CLI commands, Claude writes custom Playwright code on the fly
and executes it through a universal runner (`run.js`).

**Workflow:**

1. User describes a testing or automation need
2. Claude writes bespoke Playwright code
3. `run.js` executes it with proper module resolution
4. Browser launches visibly, returns results + screenshots

**Skill contents:**

- `SKILL.md` — concise instructions Claude reads first
- `run.js` — universal executor with module resolution
- `helpers.js` — optional utility functions
- `API_REFERENCE.md` — full Playwright API docs (loaded on demand)
- Progressive disclosure — minimal upfront, details when needed

## How it differs from agent-browser

- **agent-browser** — command-based (`click @e1`, `fill @e2 "text"`)
- **playwright-skill** — code-based (Claude writes full Playwright scripts)

The code approach gives access to Playwright's full API surface — useful
for complex multi-page flows, custom assertions, or anything the CLI
commands can't express. But heavier per-invocation.

## Why Icebox

Already using `agent-browser` CLI which covers typical browser automation
with less overhead. This would only be worth adopting if we hit limits
of the command-based approach and need full Playwright API flexibility.

## Related

- [agent-browser](../../tools/installed/2026-01-17-agent-browser.md) — current browser CLI
- [playwright-cli](../../tools/icebox/2026-02-11-playwright-cli.md) — Microsoft's CLI alternative
- [Playwright MCP plugin](../archive/2026-02-02-playwright.md) — MCP approach (archived)

## Keywords

playwright skill code-generation browser automation testing claude-code
plugin run.js custom scripts full api
