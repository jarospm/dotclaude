# Claude Code Config

Personal configuration for [Claude Code](https://claude.ai/code), managed with GNU Stow.

## What's Synced

- `CLAUDE.md` — global instructions for all projects
- `settings.json` — Claude Code settings
- `commands/` — custom slash commands
- `skills/` — reusable AI skills with reference docs
- `agents/` — specialized subagents for complex tasks

## Structure

```
dotclaude/
├── claude/                    # Stow package (symlinked to ~/)
│   └── .claude/
│       ├── CLAUDE.md
│       ├── settings.json
│       ├── commands/*.md
│       ├── skills/<name>/SKILL.md
│       └── agents/*.md
│
├── plugins/                   # Plugin tracking (not symlinked)
│   ├── installed/
│   └── icebox/
│
└── tools/                     # External tool docs (not symlinked)
    ├── installed/
    └── icebox/
```

The `claude/` directory mirrors `~/.claude/` so Stow can create symlinks correctly.

The `plugins/` and `tools/` directories are version-controlled notes — not deployed.

## Setup

Prerequisites: GNU Stow (`brew install stow`)

```bash
# Clone the repo
git clone <repo-url> ~/dotclaude
cd ~/dotclaude

# Remove existing targets (required for directory symlinks)
rm ~/.claude/CLAUDE.md ~/.claude/settings.json
rm -rf ~/.claude/commands ~/.claude/skills ~/.claude/agents

# Deploy symlinks
stow claude
```

## Stow Usage

Run from the repo root:

```bash
stow <package>        # create symlinks
stow -D <package>     # remove symlinks
stow -R <package>     # restow (remove + create)
stow -n -v <package>  # dry run (preview changes)
```

## References

- [Using GNU Stow to Manage Your Dotfiles](https://brandon.invergo.net/news/2012-05-26-using-gnu-stow-to-manage-your-dotfiles.html)

## Acknowledgments

- Code review agents (`code-simplifier`, `typescript-reviewer`) and `/code-review` command adapted from the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by Every.
