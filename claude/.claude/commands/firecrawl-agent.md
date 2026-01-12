---
description: Use Firecrawl AI agent to gather data from the web based on a natural language prompt
argument-hint: <prompt> [--url <urls>] [--schema <file>] [instructions...]
allowed-tools: Bash(firecrawl-cli agent:*), Read, Write
---

# Firecrawl Agent

Use the AI-powered Firecrawl agent to browse the web and gather data based on a natural language prompt.

## Arguments

User input: $ARGUMENTS

If no arguments provided, ask the user for the required prompt.

## Instructions

Parse the arguments to identify:

1. **Prompt** — The main task/question for the agent (required)
2. **URL(s)** — Starting URL(s) to guide the agent (optional)
3. **Schema** — JSON schema for structured output (optional)
4. **Additional instructions** — Save to file, output format, etc.

## Available Options

- `--url <urls>` — Starting URL(s), comma-separated for multiple
- `--schema <path>` — Path to JSON schema file for structured output
- `--strict` — Constrain agent strictly to provided URLs
- `--json` — Output as JSON
- `--quiet` — Suppress progress messages

## Decision Logic

**Detecting URLs in input:**

- If input contains a URL (https://...), extract it for `--url`
- Multiple URLs can be comma-separated

**Detecting schema requests:**

- If user mentions "schema" with a file path → use `--schema <path>`
- If user describes a desired structure inline → create a temp schema file first

**Detecting strict mode:**

- If user says "only from this url", "stay on site", "don't leave" → add `--strict`

**Output handling:**

- If user mentions "json" → add `--json`
- If user specifies a filename → pipe/save output to that file
- Default: display JSON output (agent results are typically structured)

## Example Inputs and Commands

**Simple prompt (no URL):**
- Input: `find the top 5 AI startups funding rounds in 2024`
- Command: `firecrawl-cli agent "find the top 5 AI startups funding rounds in 2024"`

**With URL:**
- Input: `https://news.ycombinator.com extract the top 10 stories with titles and links`
- Command: `firecrawl-cli agent "extract the top 10 stories with titles and links" --url https://news.ycombinator.com`

**With URL and save:**
- Input: `https://docs.example.com find all API endpoints, save to endpoints.json`
- Command: `firecrawl-cli agent "find all API endpoints" --url https://docs.example.com --json > endpoints.json`

**With schema file:**
- Input: `https://example.com/pricing extract pricing tiers using schema ./pricing-schema.json`
- Command: `firecrawl-cli agent "extract pricing tiers" --url https://example.com/pricing --schema ./pricing-schema.json`

**Strict mode:**
- Input: `https://example.com only from this site, find contact information`
- Command: `firecrawl-cli agent "find contact information" --url https://example.com --strict`

**Inline schema (create temp file):**
- Input: `https://example.com/team extract team members with name, role, and bio`
- Action: Create a JSON schema file with the described structure, then run with `--schema`

## Execution

1. Parse the input to identify prompt, URLs, schema, and instructions
2. If an inline schema is described, create a temporary JSON schema file
3. Build the appropriate `firecrawl-cli agent` command
4. Run the command
5. If saving to file, confirm the file was created
6. If displaying output, show the agent results
