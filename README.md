# Claude Code Workspace

Personal configuration and knowledge base for [Claude Code](https://claude.ai/code).

## What's Here

**Deployed (via GNU Stow):**
- `CLAUDE.md` — global instructions for all projects
- `settings.json` — Claude Code settings
- `commands/` — custom slash commands
- `skills/` — reusable AI skills with reference docs
- `agents/` — specialized subagents for complex tasks

**Tracked (version-controlled notes):**
- `plugins/` — MCP servers and extensions
- `tools/` — external CLI tools
- `systems/` — methodologies and frameworks
- `inbox/` — discovered resources to evaluate

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
├── tools/                     # External tool docs (not symlinked)
│   ├── installed/
│   ├── icebox/
│   ├── ecosystems/
│   └── archive/
│
├── systems/                   # Methodologies/frameworks (not symlinked)
│   ├── installed/
│   └── icebox/
│
└── inbox/                     # Discovered resources to evaluate (not symlinked)
    └── *.md                   # Flat structure with frontmatter tags
```

The `claude/` directory mirrors `~/.claude/` so Stow can create symlinks correctly.

The `plugins/`, `tools/`, `systems/`, and `references/` directories are version-controlled notes — not deployed.

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
