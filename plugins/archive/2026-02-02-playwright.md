# playwright

**Plugin:** `playwright@claude-plugins-official`
**Source:** https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/playwright
**Underlying MCP:** https://github.com/microsoft/playwright-mcp

## What it does

Browser automation MCP server by Microsoft. Enables Claude to interact with web pages through structured accessibility snapshots — no screenshots or vision models needed.

**Capabilities:**

- Navigate, click, type, fill forms, take screenshots
- Access page content via accessibility trees (fast, deterministic)
- Supports Chromium, Firefox, WebKit
- Session persistence, headless mode, device emulation
- Trace/video recording for debugging

## Why Archived

Disabled in favor of `agent-browser` CLI, which is more token-efficient
and has a richer feature set for AI agent use (semantic locators, wait
conditions, frame handling, state checks). The MCP loads 26+ tool
definitions into context on every message — expensive overhead when a
CLI tool does the same job through Bash.

If the CLI approach ever needs the MCP's strengths (persistent state,
rich introspection, iterative reasoning over page structure), consider
re-enabling.

## Related

- [agent-browser](../../tools/installed/2026-01-17-agent-browser.md) — current browser CLI (what replaced this)
- [playwright-cli](../../tools/icebox/2026-02-11-playwright-cli.md) — token-efficient CLI alternative from same team
- [playwright-mcp tool](../../tools/icebox/2026-01-17-playwright-mcp.md) — tool-level docs

## Keywords

playwright browser automation mcp microsoft web testing screenshots forms accessibility
