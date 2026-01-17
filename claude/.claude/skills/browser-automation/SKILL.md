---
name: browser-automation
description: Use agent-browser for browser automation. Use when testing web apps, scraping pages, filling forms, or automating browser interactions.
---

# Browser Automation Skill

Use `agent-browser` for web automation. Run `agent-browser --help` for all commands.

## Core Workflow

1. `agent-browser open <url>` — Navigate to page
2. `agent-browser snapshot -i` — Get interactive elements with refs (@e1, @e2)
3. `agent-browser click @e1` / `fill @e2 "text"` — Interact using refs
4. Re-snapshot after page changes

## Common Commands

```bash
# Navigation
agent-browser open "https://example.com"

# Get interactive elements (returns @e1, @e2, etc.)
agent-browser snapshot -i

# Interactions
agent-browser click @e3
agent-browser fill @e5 "search term"
agent-browser type "text to type"
agent-browser press Enter

# Visual snapshot
agent-browser screenshot ./screenshot.png
```

## Useful Options

- `--session <name>` — Isolated browser instance (parallel testing)
- `screenshot <path>` — Save visual snapshot
- `set device "iPhone 14"` — Mobile viewport emulation
- `close --session <name>` — Clean up when done

## Example: Login Flow

```bash
agent-browser open "https://app.example.com/login"
agent-browser snapshot -i
# Output shows: @e1 input[name="email"], @e2 input[name="password"], @e3 button[type="submit"]
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "password123"
agent-browser click @e3
agent-browser snapshot -i  # Re-snapshot after navigation
```

## Tips

- Always re-snapshot after clicks or navigation — refs change with DOM
- Use `--session` for parallel test scenarios
- Close sessions when done to free resources
