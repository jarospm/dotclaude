# library-docs

**Installed:** 2026-02-15
**Source:** `sawyerhood/sawyer-mart`
**Agents:** Claude Code, Codex
**Previously:** Claude Code plugin (`library-docs@sawyer-mart`), uninstalled 2026-02-15

Looks up documentation and source code for libraries by cloning repos and exploring the source. Triggers on questions like "How do I use X library?" or "What's the API for Y?"

## Install

```bash
npx skills add sawyerhood/sawyer-mart -g -a claude-code -a codex -s library-docs
```

## Notes

- Not in this repo — installed via `npx skills`, lives in `~/.agents/skills/`
- Symlinked into `~/.claude/skills/` and `~/.codex/skills/`
