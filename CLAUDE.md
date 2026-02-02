# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Configuration files for Claude Code, deployed via GNU Stow symlinks to `~/.claude/`.

## Structure

- `claude/.claude/` — Stow package, symlinked to `~/.claude/`
  - `CLAUDE.md` — global Claude Code instructions
  - `settings.json` — Claude Code settings
  - `commands/` — custom slash commands (.md files)
  - `skills/` — reusable AI skills with reference docs
  - `agents/` — specialized subagents (.md files)
- `plugins/` — plugin tracking (not symlinked)
  - `installed/` — currently active plugins
  - `icebox/` — interesting plugins for later
- `tools/` — external tool documentation (not symlinked)
  - `installed/` — tools currently in use
  - `icebox/` — interesting tools for later
  - `ecosystems/` — npx-based tooling and ecosystem knowledge
- `systems/` — opinionated systems/methodologies (not symlinked)
  - `installed/` — systems actively in use
  - `icebox/` — interesting systems for later
- `inbox/` — discovered resources to evaluate (not symlinked)
  - Flat structure with frontmatter tags for filtering

## Deployment

From repo root:

```bash
stow claude      # create symlinks
stow -D claude   # remove symlinks
stow -R claude   # restow (remove + create)
```

## Adding New Commands

- Create new .md files in claude/.claude/commands/
- Since the commands directory itself is symlinked, new files appear automatically without re-stowing

## Adding New Skills

- Create a new directory in claude/.claude/skills/<skill-name>/
- Add SKILL.md with skill definition and instructions
- Optionally add examples/ and references/ subdirectories

## Adding New Agents

- Create new .md files in claude/.claude/agents/
- Include frontmatter with `name`, `description`, `tools`, and `model`
- Agents run in isolated contexts and can be invoked by commands or directly

## Plugin Tracking

The `plugins/` directory tracks plugins (not deployed, just notes):

- `installed/` — add a file when installing a plugin
- `icebox/` — add a file for interesting plugins to try later

File naming: `YYYY-MM-DD-plugin-name.md`

## External Tools

Some commands/skills depend on external CLI tools:

- **firecrawl-cli** — `~/tools/firecrawl-cli/` — web scraping/search via Firecrawl API
  - Used by: `firecrawl-search`, `firecrawl-scrape`, `firecrawl-agent` commands
- **agent-browser** — browser automation for AI agents
  - Used by: `browser-automation` skill

See `tools/installed/` for setup instructions.

## Systems Tracking

The `systems/` directory tracks opinionated systems and methodologies — frameworks that shape how you work with AI agents, not just individual tools or plugins.

- `installed/` — systems actively integrated into workflow
- `icebox/` — interesting systems to evaluate later

File naming: `YYYY-MM-DD-system-name.md`

Examples: agent-os, cursor-rules collections, prompt engineering frameworks.

## Inbox

The `inbox/` directory is a landing zone for discovered resources — things to evaluate, try later, or promote to other folders once adopted.

File naming: `YYYY-MM-DD-short-name.md`

**Frontmatter schema:**
```yaml
---
title: Human-readable name
url: https://example.com/
type: directory | article | discussion | tool | tutorial
tags: [searchable, keywords, here]
---
```

Include a `Keywords` section in the body with search terms you might use later.

When a resource is adopted, move it to the appropriate folder (tools/installed, plugins/installed, etc.) and create proper documentation.
