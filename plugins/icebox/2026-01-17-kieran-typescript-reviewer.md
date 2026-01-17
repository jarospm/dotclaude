# kieran-typescript-reviewer

**Type:** Agent
**Added to icebox:** 2026-01-17

## Source

- GitHub: https://github.com/EveryInc/compound-engineering-plugin/blob/main/plugins/compound-engineering/agents/review/kieran-typescript-reviewer.md
- Part of: compound-engineering plugin

## Description

High-standards TypeScript code reviewer with strict quality conventions.

**For existing code:** Extremely rigorous — added complexity requires strong justification, prefers extracting to new modules over complicating existing ones.

**For new code:** More pragmatic — acceptable if isolated and functional, though improvements still flagged.

Core principles:
- **Type Safety** — never permit `any` without explanation
- **Testing as Indicator** — difficulty testing signals structural issues
- **Naming Standards** — five-second clarity rule for function/component names
- **Regression Prevention** — scrutinizes deletions for breaking changes
- **Module Extraction** — suggests new modules for multiple concerns

Philosophy: *"Duplication > Complexity"* — prefers simple duplicate code over intricate abstractions.

## Why Interested

TypeScript-specific opinionated reviewer. Could complement general code simplification.

## Notes

Alternative to [code-simplifier](2026-01-17-code-simplifier.md) (official Anthropic). TypeScript-focused with specific conventions around ES6+ patterns and type safety.
