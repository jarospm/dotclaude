---
description: Search the web using Firecrawl and optionally scrape result content
argument-hint: <query> [instructions...]
allowed-tools: Bash(firecrawl-cli search:*), Read, Write
---

# Firecrawl Search

Search the web using Firecrawl and return results.

## Arguments

$ARGUMENTS

## Instructions

Parse the arguments to identify:

1. **Query** — The search query (required)
2. **Additional instructions** — Limit, scrape content, save to file, etc.

## Available Options

- `--limit <n>` — Number of results to return
- `--scrape` — Also scrape full content from each result page
- `--lang <code>` — Language code (e.g., en, es, de)
- `--country <code>` — Country code (e.g., us, uk, de)
- `--json` — Output as JSON instead of markdown
- `--quiet` — Suppress progress messages

## Decision Logic

**Query extraction:**

- The main search terms form the query
- Wrap the query in quotes for the command

**Result limit:**

- If user mentions a number of results → add `--limit <n>`
- "top 5 results", "first 10", "limit to 3" → extract the number

**Scraping content:**

- If user wants full content, not just snippets → add `--scrape`
- Keywords: "with content", "scrape", "full text", "extract content"

**Localization:**

- If user mentions a language → add `--lang <code>`
- If user mentions a country → add `--country <code>`

**Output handling:**

- If user mentions "json" → add `--json`
- If user specifies a filename → save output to that file
- Default: formatted markdown list

## Example Inputs and Commands

**Basic search:**
- Input: `nextjs server actions best practices`
- Command: `firecrawl-cli search "nextjs server actions best practices"`

**Limited results:**
- Input: `top 5 results for rust async patterns`
- Command: `firecrawl-cli search "rust async patterns" --limit 5`

**With content scraping:**
- Input: `react hooks tutorial with full content`
- Command: `firecrawl-cli search "react hooks tutorial" --scrape`

**Save to file:**
- Input: `typescript generics guide, save to ts-generics.md`
- Command: `firecrawl-cli search "typescript generics guide" > ts-generics.md`

**JSON output:**
- Input: `firecrawl api documentation as json`
- Command: `firecrawl-cli search "firecrawl api documentation" --json`

**Localized search:**
- Input: `machine learning courses in german`
- Command: `firecrawl-cli search "machine learning courses" --lang de`

## Execution

1. Parse the input to identify query and instructions
2. Build the appropriate `firecrawl-cli search` command
3. Run the command
4. If saving to file, confirm the file was created
5. If displaying output, show the search results
