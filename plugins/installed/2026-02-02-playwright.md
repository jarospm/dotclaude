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

## Related

- [agent-browser](../../tools/installed/2026-01-17-agent-browser.md) — alternative CLI-based approach (used by browser-automation skill)
- [compound-engineering-agent-browser](../icebox/2026-01-17-compound-engineering-agent-browser.md) — different plugin, same name

## Keywords

playwright browser automation mcp microsoft web testing screenshots forms accessibility
