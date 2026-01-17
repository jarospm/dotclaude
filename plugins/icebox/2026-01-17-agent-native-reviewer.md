# agent-native-reviewer

**Type:** Agent
**Added to icebox:** 2026-01-17

## Source

- GitHub: https://github.com/EveryInc/compound-engineering-plugin/blob/main/plugins/compound-engineering/agents/review/agent-native-reviewer.md
- Part of: compound-engineering plugin
- Concept: https://every.to/guides/agent-native

## Description

Reviews code and application designs to ensure they follow "agent-native" architecture — a design philosophy from Every where AI agents are first-class users with the same capabilities as humans.

### Agent-Native Principles

- **Parity** — whatever users can do through UI, agents can do through tools
- **Granularity** — tools are atomic primitives, not choreographed workflows
- **Composability** — new capabilities through prompts, not code changes
- **Emergent Capability** — agents accomplish unanticipated tasks by combining tools
- **Shared Workspace** — agents and users operate in the same data space

### What the Reviewer Validates

- **Action Parity** — every UI action has a corresponding agent tool
- **Context Parity** — agents see the same data users see
- **Shared Workspace** — no sandboxes or sync layers between agent and user
- **Primitives over Workflows** — tools are basic operations, not encoded business logic
- **Dynamic Context Injection** — system prompts include runtime application state

## Why Interested

Useful for building apps where AI agents are first-class users. Could help catch architectural gaps early — flags issues like context starvation, orphan features, and workflow tools.

## Notes

From compound-engineering plugin (EveryInc). Specialized for agent-native app architecture review.
