# agent-browser

**Installed:** 2026-02-17
**Source:** `vercel-labs/agent-browser`
**Agents:** Claude Code, Codex
**Previously:** Custom skill in `claude/.claude/skills/browser-automation/`, removed 2026-02-17

Browser automation CLI for AI agents. Navigate pages, fill forms, click buttons, take screenshots, extract data, and test web apps.

## Install

```bash
npx skills add vercel-labs/agent-browser -g -a claude-code -a codex -s agent-browser
```

## Notes

- Not in this repo — installed via `npx skills`, lives in `~/.agents/skills/`
- Symlinked into `~/.claude/skills/` and `~/.codex/skills/`
- Replaces the custom `browser-automation` skill that was maintained in this repo
- Requires `agent-browser` CLI tool — see `tools/installed/` for setup
