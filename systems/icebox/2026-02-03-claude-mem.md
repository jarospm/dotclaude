# Claude-Mem

Persistent memory system for Claude Code — captures tool usage, generates semantic summaries, and injects relevant context into future sessions.

**Repo:** https://github.com/thedotmack/claude-mem
**Stars:** ~19.7k ⭐ | **Forks:** 1.3k | **Latest:** v9.0.12 (Jan 2026)

## What It Does

Claude Code plugin that preserves context across sessions:

- Automatically captures observations during coding work
- Compresses observations using AI summarization
- Stores in SQLite + Chroma vector database
- Injects relevant past context when starting new sessions
- Enables natural language search of project history

## Architecture

**7 Hook Scripts (6 lifecycle + 1 pre-hook):**
- Smart Install — cached dependency checker (pre-hook)
- SessionStart — injects relevant context
- UserPromptSubmit — captures user intent
- PostToolUse — records tool observations
- Stop — handles interruptions
- SessionEnd — finalizes session data

**Infrastructure:**
- Worker service (Bun runtime) on port 37777
- SQLite for sessions, observations, summaries
- Chroma vector database for hybrid semantic + keyword search
- Web viewer UI at `http://localhost:37777`

**4 MCP Tools — 3-Layer Pattern:**
1. `search` — compact index with IDs (~50-100 tokens per result)
2. `timeline` — chronological context around results
3. `get_observations` — full details for filtered IDs (~500-1000 tokens)
4. `__IMPORTANT` — workflow documentation (always visible to Claude)

This progressive disclosure approach claims ~10x token savings by filtering before fetching.

## Installation

```bash
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

Requires Node.js 18+, auto-installs Bun if needed.

## Key Features

- **Privacy Control** — `<private>` tags exclude sensitive content
- **Citation System** — reference past observations by ID
- **Progressive Disclosure** — layered retrieval with visible token costs
- **Automatic Operation** — works without manual intervention
- **Claude Desktop Support** — search memory from Desktop conversations
- **Troubleshoot Skill** — automated diagnostics for issues
- **Endless Mode** — beta feature with "biomimetic memory architecture" for extended sessions

## Configuration

Settings in `~/.claude-mem/settings.json`:
- AI model selection
- Worker port
- Data directory
- Logging level
- Context injection behavior

## License

- Main code: AGPL-3.0
- `ragtime/` directory: PolyForm Noncommercial License 1.0.0

## Why Interesting

- Solves the "fresh context" problem in Claude Code sessions
- Uses vector search for semantic retrieval — not just keyword matching
- Token-efficient progressive disclosure pattern
- Privacy-aware with explicit exclusion tags
- Could complement or replace manual memory management approaches

## Community

- **Discord:** https://discord.com/invite/J4wttp9vDu
- **X/Twitter:** [@Claude_Memory](https://x.com/Claude_Memory)
- **Translations:** 24 languages

## Considerations

- AGPL license requires sharing modifications
- Runs a background worker service (port 37777)
- Depends on Bun runtime + uv (Python package manager)
- `ragtime/` component has noncommercial restrictions
- **$CMEM crypto token** — project has a Solana token (unusual for dev tools)
- Potential overlap with Claude Code's built-in memory features

## Status

Saved for evaluation. Very popular (~20k stars) and actively maintained. Interesting approach to persistent memory that goes beyond simple file-based solutions. The crypto token is a red/yellow flag worth investigating.
