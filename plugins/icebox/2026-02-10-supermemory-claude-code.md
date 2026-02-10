# supermemory-claude-code

URL: https://github.com/supermemoryai/claude-supermemory
Docs: https://supermemory.ai/docs/intro
Tweet: https://x.com/DhravyaShah/status/2017039283367137690

## What it does

Claude Code plugin that gives persistent memory across sessions.
Replaces the "Groundhog Day" problem of re-explaining context
every session.

**How it works:**

- **Context injection** — on session start, injects a User Profile
  built from prior interactions
- **Automatic capture** — conversation turns are stored for
  future context without manual saving
- **Hybrid memory** — not just RAG similarity search; extracts
  facts, tracks how they change over time, builds an evolving
  user profile

**Why plugin over MCP:**

The MCP server exists too, but the plugin adds two things the
MCP can't do — automatic context injection at session start,
and automatic conversation capture. With MCP alone, Claude
decides when to call the tools (unreliable for learning).

**Benchmarks:**

- 81.6% on LongMemEval (vs 40-60% for typical RAG)

**Considerations:**

- Sends conversation data to Supermemory's cloud service
- Privacy/security implications for proprietary codebases
- Overlaps with CLAUDE.md approach but more automated

## Keywords

supermemory memory context persistence claude-code plugin
long-term-memory rag user-profile session dhravya
