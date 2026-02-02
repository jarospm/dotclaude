# Compound Engineering

Workflow system where each unit of work makes subsequent work easier.

**Repo:** https://github.com/EveryInc/compound-engineering-plugin

## What It Does

Four-command cycle:
- `/workflows:plan` — convert ideas into detailed implementation plans
- `/workflows:work` — execute plans with worktrees and task tracking
- `/workflows:review` — multi-agent code review before merging
- `/workflows:compound` — document learnings for future reuse

## Philosophy

- "80% planning and review, 20% execution"
- Prevent technical debt through deliberate process
- Capture knowledge so it becomes reusable infrastructure
- Each project should make the next one easier

## Plugin Contents

**Skills:**
- `agent-browser` — browser automation skill (not to be confused with [Vercel's agent-browser CLI](../../tools/installed/2026-01-17-agent-browser.md))

**Agents:**
- `agent-native-reviewer` — reviews code for "agent-native" architecture
  - Validates action parity, context parity, shared workspace
  - Based on [agent-native philosophy](https://every.to/guides/agent-native)

## Integration

- Installed via Claude Code plugin system
- MIT licensed
- Maintained by EveryInc

## Why Interesting

- Explicit focus on knowledge compounding
- Multi-agent review stage
- Similar to superpowers but emphasizes learning capture

## Status

Saved for later evaluation. Compare with superpowers and agent-os.
