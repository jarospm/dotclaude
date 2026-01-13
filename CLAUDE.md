# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Configuration files for Claude Code, deployed via GNU Stow symlinks to `~/.claude/`.

## Structure

- `claude/.claude/CLAUDE.md` — global Claude Code instructions (symlinked to ~/.claude/)
- `claude/.claude/settings.json` — Claude Code settings
- `claude/.claude/commands/` — custom slash commands (.md files)
- `claude/.claude/skills/` — reusable AI skills with reference docs

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
