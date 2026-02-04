---
title: Ralph Loop
url: https://github.com/snarktank/ralph
origin: Geoffrey Huntley (ghuntley.com/loop/)
popularized_by: Ryan Carson (@ryancarson)
tags: [autonomous-agents, ai-coding, loop-pattern, context-management]
---

# Ralph Loop

Autonomous AI agent loop that runs coding tools (Claude Code, Amp) repeatedly until all PRD items are complete.

## Core Philosophy

- **One context window, one task, one goal** — each iteration is a fresh process
- **Memory lives in files, not conversation** — git commits, progress.txt, prd.json
- **Context pollution is the enemy** — failed attempts accumulate and confuse the model
- **Fresh context cannot degrade** — starts clean every time, no compaction loss

## How It Works

1. Create PRD with granular user stories (each completable in one context window)
2. Bash script launches AI tool with customized prompt
3. AI picks highest-priority incomplete story
4. Quality gates run (typecheck, tests)
5. On success: mark complete, update learnings, commit
6. Loop repeats until PRD complete

## Key Principles

- **Specification matters** — spend 30+ minutes on user stories before kicking off
- **Watch the loop** — observe thinking and decisions, this is where you learn
- **Tune with "signs"** — add constraints to prompts when behavior drifts
- **Monolithic by design** — single repo, single process, one task per loop

## Common Mistakes

- Using single persistent context (degrades via compaction)
- Skipping conversation phase before spec generation
- Stories too large for one context window
- Running silently instead of observing

## Implementation Variants

Three main implementations exist, each building on the previous:

### Geoffrey Huntley (Original Concept)

The philosophy, not a specific tool. Key ideas:

- **Monolithic by design** — single repo, single process, one task per loop
- **"Clay on the pottery wheel"** — when something fails, return to the loop
- Manual prompting or automation with strategic pauses (Ctrl+C between tasks)
- Emphasis on watching the loop to identify failure domains
- Context engineering over conversation persistence

### Ryan Carson's Ralph (snarktank/ralph)

First practical implementation of Huntley's concept:

- **Fresh context per iteration** — spawns new AI instance each cycle
- **Memory through files** — git commits, progress.txt, prd.json, learnings file
- **Single-agent execution** — one task at a time, sequential
- **Quality gates** — typecheck/tests run between iterations
- **AGENTS.md integration** — accumulates learnings for future iterations
- Supports Amp CLI and Claude Code

### Michael Shimeles's Ralphy

Evolution adding parallel coordination and broader tooling:

- **Parallel execution** — run 3+ agents simultaneously with isolated worktrees
- **AI-assisted merge resolution** — auto-resolves conflicts when branches merge
- **Many AI engines** — Claude Code, OpenCode, Cursor, Codex, Qwen-Code, Factory Droid, GitHub Copilot, Gemini CLI
- **Browser automation** — agent-browser integration for UI verification
- **Rich task sources** — Markdown, YAML/JSON, folders, GitHub issues
- **Webhooks/notifications** — Discord, Slack integration
- **Project config** — `.ralphy/config.yaml` for rules, boundaries, test commands

**Key difference**: Carson's Ralph is sequential single-agent; Shimeles's Ralphy adds parallel multi-agent orchestration with conflict resolution.

## Related Systems

Ralph is the execution layer in a larger stack:

1. **Ralph** (this) — autonomous loop pattern, fresh context per task
2. **Compound Engineering** — methodology for extracting/persisting learnings
3. **Compound Product** — automation layer scaling to product/org level

See Ryan Carson's synthesis: [How to make your agent learn and ship while you sleep](https://x.com/ryancarson/status/2016520542723924279)

## Resources

- Original concept: https://ghuntley.com/loop/
- Ryan Carson's repo: https://github.com/snarktank/ralph
- Michael Shimeles's Ralphy: https://github.com/michaelshimeles/ralphy
- Deep dive: https://thetrav.substack.com/p/the-real-ralph-wiggum-loop-what-everyone
