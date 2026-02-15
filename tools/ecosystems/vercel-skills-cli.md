# Vercel Skills CLI

The open agent skills ecosystem — "npm for AI agents."

## Source

- GitHub: https://github.com/vercel-labs/skills
- Registry: https://skills.sh

## What It Is

A CLI tool (`npx skills`) for installing, managing, and updating reusable AI agent skills. Supports 39+ agents including Claude Code, Codex, Cursor, Copilot, and others.

Skills are `SKILL.md` files with YAML frontmatter that extend agent capabilities.

## Architecture

```
~/.agents/skills/              # shared source of truth
  find-skills/
  vercel-react-best-practices/
  web-design-guidelines/

~/.claude/skills/              # Claude Code (real dir via --no-folding)
  convex-development/          # Stow-managed (custom skill)
  firecrawl/                   # Stow-managed (custom skill)
  find-skills → ../../.agents  # npx skills-managed (symlink)
  vercel-react-best-practices → ../../.agents  # npx skills-managed

~/.codex/skills/               # Codex (universal integration)
  find-skills → ../../.agents
```

Two types of skills coexist in `~/.claude/skills/`:

- **Custom skills** — real directories managed by Stow from `dotclaude/claude/.claude/skills/`
- **Community skills** — symlinks to `~/.agents/skills/` managed by `npx skills`

## Key Commands

```bash
npx skills add <owner/repo>          # install skill from GitHub
npx skills add <owner/repo> -g       # install globally
npx skills add <owner/repo> --list   # list available skills in repo
npx skills remove <skill-name>       # remove a skill
npx skills list                      # list installed skills
npx skills list -g                   # list global skills
npx skills check                     # check for updates
npx skills update                    # update all skills
npx skills find <query>              # search for skills
```

## Key Flags

- `-a, --agent <agents...>` — target specific agents (repeat flag per agent)
- `-s, --skill <skills...>` — install specific skills by name
- `-g, --global` — install to global location
- `-y, --yes` — skip confirmation prompts
- `--copy` — copy files instead of symlinking
- `--list` — list available skills in a repo before installing

## Installing Skills

Always scope to specific agents with `-a` to avoid cluttering 30+ agent directories:

```bash
# Install a specific skill for Claude Code and Codex
npx skills add vercel-labs/agent-skills -g -a claude-code -a codex -s vercel-react-best-practices

# Install all skills from a repo
npx skills add vercel-labs/agent-skills -g -a claude-code -a codex -s '*'

# List what's in a repo before installing
npx skills add vercel-labs/agent-skills --list
```

**Important:** the source argument must come right after `add`, before flags.

## Installation Methods

- **Symlink (default)** — Claude Code gets symlinks in `~/.claude/skills/` pointing to `~/.agents/skills/`
- **Universal** — some agents (Codex) read from `~/.agents/skills/` natively
- **Copy (`--copy`)** — independent copies per agent, must update each separately

## How Updates Work

The CLI maintains `~/.agents/.skill-lock.json` tracking source repos and folder hashes.

- `npx skills check` — compares local hash to GitHub's current tree SHA
- `npx skills update` — pulls latest from source repos
- No semantic versioning — tracks latest commit

## Stow Integration (Resolved)

This repo manages `~/.claude/` via GNU Stow. To let custom skills and community skills coexist:

1. **Stow with `--no-folding`** — makes `~/.claude/skills/` a real directory (not a folder symlink)
2. **Custom skills** — Stow creates individual symlinks for each file in each skill
3. **Community skills** — `npx skills` creates symlinks to `~/.agents/skills/`
4. **New custom skills require restow** — `stow -R --no-folding claude`
5. **New community skills** — just run `npx skills add` with `-a claude-code`

## Tracking

Installed community skills are tracked in `skills/installed/` with date-prefixed pages. Use `npx skills list -g` for the live source of truth.
