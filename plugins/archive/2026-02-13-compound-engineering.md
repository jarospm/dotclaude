# compound-engineering

**Type:** Skills, Commands, Agents
**Added:** 2026-02-13
**Marketplace:** every-marketplace
**Source:** https://github.com/EveryInc/compound-engineering-plugin
**Previously:** systems/icebox (evaluated 2026-01-21)

## Installation

```bash
/plugin marketplace add https://github.com/EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

## Purpose

Workflow plugin where each unit of engineering work makes subsequent work easier. Follows a Plan → Work → Review → Compound cycle where learnings from each cycle inform future ones.

## Philosophy

- "80% planning and review, 20% execution"
- Prevent technical debt through deliberate process
- Capture knowledge so it becomes reusable infrastructure

## Key Commands

- `/workflows:plan` — transform feature descriptions into structured plans
- `/workflows:work` — execute plans with task tracking and git worktrees
- `/workflows:review` — multi-agent code review
- `/workflows:compound` — document learnings for future reuse
- `/workflows:brainstorm` — explore requirements before planning

## Bundled Skills and Agents

- `agent-browser` — browser automation skill
- `agent-native-reviewer` — reviews code for "agent-native" architecture
- `frontend-design` — UI generation skill
- `git-worktree` — isolated parallel development
