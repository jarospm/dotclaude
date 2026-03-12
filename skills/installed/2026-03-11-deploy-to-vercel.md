# deploy-to-vercel

**Installed:** 2026-03-11
**Source:** `vercel-labs/agent-skills`
**Agents:** Claude Code

Deploy applications and websites to Vercel. Preview deployments by default, production only when explicitly requested.

## Install

```bash
npx skills add https://github.com/vercel-labs/agent-skills -g -a claude-code -s deploy-to-vercel
```

## Notes

- Not in this repo — installed via `npx skills`, lives in `~/.agents/skills/`
- Symlinked into `~/.claude/skills/`
- Requires `vercel` CLI installed and authenticated (`vercel whoami`)
- Tries to get the project into a linked state with git-push deploys
