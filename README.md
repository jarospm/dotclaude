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
- `skills/` — community skills installed via `npx skills`
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
├── skills/                    # Community skill tracking (not symlinked)
│   └── installed/             # Skills installed via `npx skills`
│
├── plugins/                   # Plugin tracking (not symlinked)
│   ├── installed/
│   └── icebox/
│
├── tools/                     # External tool docs (not symlinked)
│   ├── installed/
│   ├── icebox/
│   └── ecosystems/
│
├── systems/                   # Methodologies/frameworks (not symlinked)
│   ├── installed/
│   └── icebox/
│
└── inbox/                     # Discovered resources to evaluate (not symlinked)
    └── *.md                   # Flat structure with frontmatter tags
```

The `claude/` directory mirrors `~/.claude/` so Stow can create symlinks correctly.

The `skills/`, `plugins/`, `tools/`, `systems/`, and `inbox/` directories are version-controlled notes — not deployed.

## Setup

Prerequisites: GNU Stow (`brew install stow`)

```bash
# Clone the repo
git clone <repo-url> ~/dotclaude
cd ~/dotclaude

# Remove existing targets (required for first-time setup)
rm ~/.claude/CLAUDE.md ~/.claude/settings.json
rm -rf ~/.claude/commands ~/.claude/skills ~/.claude/agents

# Deploy symlinks
stow --no-folding claude
```

**Why `--no-folding`?** Without it, Stow symlinks entire directories (e.g. `~/.claude/skills/` → repo). With `--no-folding`, Stow creates real directories and symlinks individual files. This lets community skills from `npx skills` coexist alongside custom skills in `~/.claude/skills/`.

## Stow Usage

Run from the repo root:

```bash
stow --no-folding claude        # create symlinks
stow -D claude                  # remove symlinks
stow -R --no-folding claude     # restow (remove + create)
stow -n -v --no-folding claude  # dry run (preview changes)
```

## Community Skills (`npx skills`)

The [Skills CLI](https://github.com/vercel-labs/skills) (`npx skills`) is a package manager for the open agent skills ecosystem. Skills are `SKILL.md` files that extend agent capabilities.

**Architecture:**

- `~/.agents/skills/` — shared source of truth, managed by `npx skills`
- `~/.claude/skills/` — Claude Code reads from here. Contains both:
  - Custom skills — real directories, Stow-managed from this repo
  - Community skills — symlinks to `~/.agents/skills/`

**Installing a skill:**

```bash
# Source argument comes right after `add`, flags after
npx skills add vercel-labs/agent-skills -g -a claude-code -a codex -s vercel-react-best-practices

# List what's in a repo before installing
npx skills add vercel-labs/agent-skills --list

# See all installed skills
npx skills list -g

# Check for updates / update
npx skills check
npx skills update
```

Always use `-a claude-code -a codex` (repeated flags, not comma-separated) to avoid installing into 30+ agent directories you don't use.

See `tools/ecosystems/vercel-skills-cli.md` for full reference.

## References

- [Using GNU Stow to Manage Your Dotfiles](https://brandon.invergo.net/news/2012-05-26-using-gnu-stow-to-manage-your-dotfiles.html)

## Acknowledgments

- Code review agents (`code-simplifier`, `typescript-reviewer`) and `/code-review` command adapted from the [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) by Every.
