# agent-browser

Browser automation CLI for AI agents — interact with web pages via snapshots and element refs.

## Source

- Website: https://agent-browser.dev/
- GitHub: https://github.com/vercel-labs/agent-browser

## Used By

- `skills/browser-automation/SKILL.md`

## Installation

```bash
npm install -g agent-browser
```

## Quick Reference

```bash
agent-browser open <url>       # Navigate to page
agent-browser snapshot -i      # Get interactive elements (@e1, @e2, etc.)
agent-browser click @e1        # Click element by ref
agent-browser fill @e2 "text"  # Fill input field
agent-browser type "text"      # Type text
agent-browser press Enter      # Press key
agent-browser screenshot ./ss.png
agent-browser close            # Close browser
```

## Core Workflow

1. `open` a URL
2. `snapshot -i` to get interactive elements with refs
3. `click` or `fill` using refs (@e1, @e2)
4. Re-snapshot after page changes (refs update with DOM)

## Options

- `--session <name>` — isolated browser instance for parallel testing
- `set device "iPhone 14"` — mobile viewport emulation

## Full Documentation

See the skill file at `skills/browser-automation/SKILL.md` or visit https://agent-browser.dev/
