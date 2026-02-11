# playwright-cli

Token-efficient browser automation CLI for AI coding agents — part of Microsoft's Playwright MCP project.

## Source

- GitHub: https://github.com/microsoft/playwright-mcp (CLI lives in same repo as MCP)
- Skills reference: https://skills.sh/microsoft/playwright/playwright-cli
- Maintained by: Microsoft

## Why It Exists

The Playwright MCP server loads 26+ tool definitions and full accessibility
trees into context on every action — expensive for coding agents. The CLI
mode avoids this by exposing the same Playwright engine through lightweight
Bash commands that return only what you ask for.

## Installation

```bash
npx @playwright/mcp@latest --cli
```

Requires Node.js 18+.

## Commands

### Core Interaction

- `open [url] [--headed] [--browser=chrome|firefox|webkit|msedge]`
- `goto <url>`
- `click <ref>` / `dblclick <ref>` / `hover <ref>`
- `fill <ref> "text"` / `type "text"`
- `select <ref> <value>`
- `check <ref>` / `uncheck <ref>`
- `drag <source-ref> <target-ref>`
- `upload <ref> <file-path>`
- `press <key>` (Enter, Tab, Control+a, etc.)
- `keydown <key>` / `keyup <key>`
- `snapshot` — capture page state with element refs (`e1`, `e2`)
- `eval <js>` — execute JavaScript on page
- `close`

### Navigation

- `go-back` / `go-forward` / `reload`

### Mouse Control

- `mousemove <x> <y>`
- `mousedown` / `mouseup`
- `mousewheel <dx> <dy>`

### Screenshots & Recording

- `screenshot [path]`
- `pdf [path]`
- `video-start` / `video-stop` — record browser session
- `tracing-start` / `tracing-stop` — Playwright trace viewer format

### Tab Management

- `tab-list` / `tab-new` / `tab-close <index>` / `tab-select <index>`

### Storage & Cookies

- `cookie-list` / `cookie-get <name>` / `cookie-set <name> <value>` / `cookie-delete <name>` / `cookie-clear`
- `localstorage-get <key>` / `localstorage-set <key> <value>` / `localstorage-clear`
- `sessionstorage-get <key>` / `sessionstorage-set <key> <value>` / `sessionstorage-clear`
- `state-save <path>` / `state-load <path>` — full browser state snapshots

### Network

- `route <url-pattern> <response-json>` — intercept/mock requests
- `unroute <url-pattern>`
- `route-list` — view active routes
- `network` — monitor requests
- `console` — view console messages

### Dialogs

- `dialog-accept [text]` / `dialog-dismiss`

### Viewport & Emulation

- `resize <width> <height>`
- Supports `--browser` flag for multi-browser testing

### Session Management

- `-s=<name>` — named session (isolated browser instance)
- `list` — view active sessions
- `close-all` / `kill-all` — terminate all browsers
- Persistent profile by default (cookies survive between calls)
- `--persistent` flag for explicit persistent profile

### Debugging

- `run-code` — execute arbitrary Playwright code
- YAML interaction recording — auto-records actions for replay

## How It Compares to agent-browser

Both are CLI wrappers around `playwright-core` for AI agents.

**playwright-cli advantages:**

- Video recording and tracing (agent-browser lacks this)
- `run-code` for arbitrary Playwright execution
- Multi-browser support via `--browser` flag
- YAML action recording for test generation
- `state-save` / `state-load` for explicit state snapshots
- Persistent profile by default
- Microsoft-backed, same team as Playwright itself

**agent-browser advantages:**

- Rust binary — faster startup and execution
- Richer semantic locators (ARIA role, text, label, placeholder, alt, testid)
- Element state checking (`is visible`, `is enabled`, `is checked`)
- Element data extraction (text, innerHTML, attributes, bounding boxes)
- Frame/iframe handling
- Rich wait conditions (element, text, URL, network idle, JS condition)
- Page error tracking (uncaught exceptions)
- Element highlighting for visual debugging

**Verdict:** agent-browser is better for typical AI agent browser automation
(richer inspection, better waiting, semantic locators). playwright-cli is
better for debugging and recording workflows (video, tracing, YAML replay).

## When to Consider Adopting

- If you need video recording of browser sessions
- If you need Playwright trace viewer integration
- If you need multi-browser testing (Firefox, WebKit, Edge)
- If agent-browser is discontinued or falls behind
- If Microsoft adds features that close the inspection/waiting gap

## Related

- [agent-browser](../installed/2026-01-17-agent-browser.md) — current browser CLI (Vercel Labs)
- [playwright-mcp](2026-01-17-playwright-mcp.md) — MCP server version (token-heavy, icebox'd)
- [Playwright MCP plugin](../../plugins/archive/2026-02-02-playwright.md) — Claude Code plugin (disabled)

## Keywords

playwright cli browser automation microsoft token-efficient video recording
tracing coding agents ai testing multi-browser webkit firefox
