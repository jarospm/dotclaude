---
title: Compound Product
url: https://github.com/snarktank/compound-product
author: Ryan Carson (@ryancarson)
related: [compound-engineering, ralph-loop]
tags: [autonomous-agents, product-ops, continuous-improvement, automation]
---

# Compound Product

Automation system that continuously improves software by analyzing performance reports and autonomously implementing fixes via PRs.

## Relationship to Compound Engineering

Compound Engineering is a methodology/plugin for individual work sessions.
Compound Product scales those principles to product/organization level — automated continuous improvement loops.

## How It Works

Four-phase cycle:

1. **Report generation** — your existing systems produce performance/error reports
2. **LLM analysis** — identifies the #1 highest-priority actionable item
3. **Branch + plan** — AI agent creates implementation plan
4. **Execution loop** — iterative implementation with quality checks

All changes land as PRs for human review — no direct merges.

## The Two-Part Nightly Loop

From Carson's synthesis article — run two jobs in sequence:

**10:30 PM — Compound Review**
- Reviews all threads from the last 24 hours
- Extracts missed learnings retroactively
- Updates relevant AGENTS.md files with patterns, gotchas, context
- Commits and pushes to main

**11:00 PM — Auto-Compound**
- Pulls latest (now with fresh learnings from review job)
- Picks #1 priority from reports
- Creates PRD, breaks into tasks, executes via Ralph loop
- Opens draft PR

The order matters: review job updates AGENTS.md with patterns discovered during the day; implementation job then benefits from those learnings.

## The Compound Effect

> "The agent gets smarter every day because it's reading its own updated instructions before each implementation run. Patterns discovered on Monday inform Tuesday's work. Gotchas hit on Wednesday are avoided on Thursday."

> "The goal is a self-improving loop: every unit of work makes future work easier. Your AGENTS.md files become institutional memory. Your agent becomes an expert in *your* codebase."

**Philosophy:** "Stop prompting. Start compounding."

## Key Features

- **Autonomous prioritization** — reads reports, picks what matters most
- **Memory persistence** — updates AGENTS.md and progress logs between iterations
- **Quality gates** — configurable checks run before commits
- **Human oversight** — PR-based workflow keeps humans in the loop
- **Flexible LLM** — Anthropic, OpenRouter, or custom gateways

## Structure

- `scripts/compound/` — core automation scripts
- `skills/` — reusable AI agent capabilities (PRD creation, task generation)
- `examples/` — configuration templates, scheduling examples

## Why Interesting

- Working implementation, not just methodology docs
- Scales compound engineering from session → organization
- Combines Ralph-style loops with performance-driven prioritization
- Same author as Ralph repo (Ryan Carson / snarktank)

## Related Systems

Part of a three-layer stack:

1. **Ralph** — autonomous loop pattern (execution layer)
2. **Compound Engineering** — methodology for extracting/persisting learnings
3. **Compound Product** (this) — automation layer scaling to product/org level

## Resources

- Repo: https://github.com/snarktank/compound-product
- Synthesis article: [How to make your agent learn and ship while you sleep](https://x.com/ryancarson/status/2016520542723924279)

## Status

Saved for later evaluation. Compare with compound-engineering and ralph-loop.
