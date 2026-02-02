# Vercel Skills CLI

The open agent skills ecosystem — "npm for AI agents."

## Source

- GitHub: https://github.com/vercel-labs/skills
- Registry: https://skills.sh

## What It Is

A CLI tool (`npx skills`) for installing, managing, and updating reusable AI agent skills. Supports 39+ agents including Claude Code, Cursor, Copilot, Codex, and others.

Skills are `SKILL.md` files with YAML frontmatter that extend agent capabilities.

## Key Commands

```bash
npx skills add <owner/repo>          # Install skill from GitHub
npx skills add <owner/repo> -g       # Install globally (~/.claude/skills/)
npx skills add <owner/repo> --list   # List available skills in repo
npx skills remove <skill-name>       # Remove a skill
npx skills list                      # List installed skills
npx skills list -g                   # List global skills
npx skills check                     # Check for updates
npx skills update                    # Update all skills
npx skills find <query>              # Search for skills
```

## Key Flags

```bash
-a, --agent <agents...>    # Target specific agents (claude-code, codex, cursor, etc.)
-s, --skill <skills...>    # Install specific skills by name (use '*' for all)
-g, --global               # Install to global location (~/.claude/skills/)
--copy                     # Copy files instead of symlinking
--list                     # List available skills in a repo before installing
```

**Common usage:**

```bash
# Install firecrawl skill only for Claude Code
npx skills add firecrawl/cli -a claude-code

# Install specific skill from a multi-skill repo
npx skills add vercel-labs/agent-skills -s frontend-design -a claude-code

# Install globally for Claude Code only
npx skills add firecrawl/cli -a claude-code -g
```

## How Updates Work

The CLI maintains `~/.agents/.skill-lock.json` with:

```json
{
  "skill-name": {
    "name": "skill-name",
    "source": "owner/repo",
    "skillFolderHash": "<GitHub tree SHA>"
  }
}
```

- `npx skills check` compares local hash to GitHub's current tree SHA
- `npx skills update` pulls latest from source repos
- No semantic versioning — tracks latest commit

## Installation Methods

**Symlink (default, recommended):**

- Single source of truth
- Updates propagate to all agents automatically

**Copy (`--copy` flag):**

- Independent copies per agent
- Must update each copy separately

## Stow Conflict

This repo manages `~/.claude/` via GNU Stow symlinks. The skills CLI expects `~/.claude/skills/` to be a mutable directory it can write to.

**Options:**

1. **Use project-local skills** (no `-g` flag) — keeps them out of Stow-managed directory
2. **Install in this repo** — run `npx skills add <skill>` from repo root, commit result
3. **Accept mixed management** — Stow for custom skills, CLI for ecosystem skills
4. **Manual updates** — copy SKILL.md manually, no auto-updates

If using option 2 or 4, the lock file won't track the skill, so `npx skills check/update` won't work.
