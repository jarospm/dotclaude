# playwright-mcp

MCP server for browser automation via Playwright's accessibility tree.

## Source

- GitHub: https://github.com/microsoft/playwright-mcp
- Maintained by: Microsoft

## Description

Model Context Protocol server that enables LLMs to automate web browsers using Playwright. Uses accessibility tree instead of screenshots — fast, deterministic, no vision models needed.

### Key Features

- **Accessibility tree based** — operates on structured data, not pixels
- **LLM-friendly** — no vision models required
- **Deterministic** — avoids ambiguity of screenshot-based approaches

## Installation

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

Requires Node.js 18+.

## Why Interested

Alternative to agent-browser. Microsoft-backed, uses accessibility tree which may be more reliable than snapshot-based approaches. Could replace or complement current browser automation setup.

## Notes

Compare with current agent-browser setup before switching.
