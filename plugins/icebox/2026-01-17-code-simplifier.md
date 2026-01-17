# code-simplifier

**Type:** Agent
**Added to icebox:** 2026-01-17
**Marketplace:** claude-plugins-official

## Source

- GitHub: https://github.com/anthropics/claude-plugins-official/blob/main/plugins/code-simplifier/agents/code-simplifier.md

## Description

Agent that simplifies and refines code for clarity, consistency, and maintainability while preserving functionality. Works autonomously after code modifications.

Key behaviors:
- Preserves functionality — only improves *how*, not *what*
- Reduces complexity and nesting
- Eliminates redundant code
- Improves naming
- Avoids nested ternaries
- Follows project conventions

## Why Interested

Could help maintain code quality automatically during development sessions.

## Notes

Official Anthropic plugin. Runs proactively after modifications — might be too aggressive for some workflows.

Alternatives from compound-engineering:
- [code-simplicity-reviewer](2026-01-17-code-simplicity-reviewer.md) — on-demand, focuses on YAGNI
- [kieran-typescript-reviewer](2026-01-17-kieran-typescript-reviewer.md) — TypeScript-specific, opinionated
