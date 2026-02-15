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
- `skills/` — community skill tracking (not symlinked)
  - `installed/` — skills installed via `npx skills`
- `plugins/` — plugin tracking (not symlinked)
  - `installed/` — currently active plugins
  - `icebox/` — interesting plugins for later
  - `archive/` — previously used plugins, consciously disabled
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
stow --no-folding claude      # create symlinks
stow -D claude                # remove symlinks
stow -R --no-folding claude   # restow (remove + create)
```

**Why `--no-folding`?** Without it, Stow creates folder symlinks (e.g. `~/.claude/skills/` → repo dir). With `--no-folding`, Stow creates real directories and symlinks individual files. This lets `npx skills` add agent skills alongside custom skills in `~/.claude/skills/`.

## Adding New Commands

- Create new .md files in `claude/.claude/commands/`
- Run `stow -R --no-folding claude` to pick up changes

## Adding New Skills

Custom skills (this repo):

- Create a new directory in `claude/.claude/skills/<skill-name>/`
- Add SKILL.md with skill definition and instructions
- Optionally add `examples/` and `references/` subdirectories
- Run `stow -R --no-folding claude` to pick up changes

Community skills (via `npx skills`):

- `~/.agents/skills/` is the shared source of truth across all agents
- `npx skills` creates symlinks from `~/.claude/skills/` → `~/.agents/skills/`
- Always scope to specific agents with repeated `-a` flags:
  ```bash
  npx skills add vercel-labs/agent-skills -g -a claude-code -a codex -s <skill-name>
  ```
- Source argument must come right after `add`, before flags
- Use `npx skills list -g` to see all installed skills
- Use `npx skills check` / `npx skills update` to update
- Track installed skills in `skills/installed/` with date-prefixed pages
- See `tools/ecosystems/vercel-skills-cli.md` for full CLI reference

## Adding New Agents

- Create new .md files in `claude/.claude/agents/`
- Include frontmatter with `name`, `description`, `tools`, and `model`
- Agents run in isolated contexts and can be invoked by commands or directly
- Run `stow -R --no-folding claude` to pick up changes

## Plugin Tracking

The `plugins/` directory tracks plugins (not deployed, just notes):

- `installed/` — add a file when installing a plugin
- `icebox/` — add a file for interesting plugins to try later
- `archive/` — move here when disabling a previously used plugin

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
