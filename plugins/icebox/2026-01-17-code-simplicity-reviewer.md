# code-simplicity-reviewer

**Type:** Agent
**Added to icebox:** 2026-01-17

## Source

- GitHub: https://github.com/EveryInc/compound-engineering-plugin/blob/main/plugins/compound-engineering/agents/review/code-simplicity-reviewer.md
- Part of: compound-engineering plugin

## Description

Reviews code to identify and eliminate unnecessary complexity. Designed for post-implementation review before finalizing changes.

Key focus areas:
- **Line-by-line scrutiny** — questions whether each element serves current requirements
- **Logic simplification** — transforms complex conditionals, reduces nesting
- **Redundancy elimination** — consolidates duplicate patterns
- **Abstraction evaluation** — challenges premature generalizations
- **YAGNI enforcement** — removes speculative "just in case" code
- **Readability** — favors self-documenting code over comments

## Why Interested

Post-implementation simplification pass. Estimates potential code reduction.

## Notes

Alternative to [code-simplifier](2026-01-17-code-simplifier.md) (official Anthropic). This one is invoked on-demand rather than running proactively.
