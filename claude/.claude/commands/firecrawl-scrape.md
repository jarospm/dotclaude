---
description: Scrape a URL using firecrawl-cli and optionally save or process the content
argument-hint: <url> [instructions...]
allowed-tools: Bash(firecrawl-cli scrape:*), Read, Write
---

# Firecrawl Scrape

Scrape a URL using `firecrawl-cli` and handle the output according to user instructions.

## Arguments

$ARGUMENTS

## Instructions

Parse the arguments to extract:

1. **URL** — The first argument should be the URL to scrape
2. **Additional instructions** — Any text after the URL describes what to do

## Available Options

- `--formats <list>` — Content formats: markdown, html, links, screenshot (default: markdown)
- `--only-main` — Extract only main content, excluding navigation/headers/footers
- `--wait <ms>` — Additional wait time for JavaScript rendering
- `--json` — Output as JSON instead of markdown
- `--quiet` — Suppress progress messages

## Decision Logic

Based on the user's instructions, determine:

**Output format:**

- If user mentions "json" → add `--json`
- If user wants clean article content → add `--only-main`
- Default: plain markdown

**Saving to file:**

- If user specifies a filename → pipe output to that file
- If no filename → display output directly

**Examples of instruction parsing:**

- "save to docs/article.md" → `firecrawl-cli scrape <url> > docs/article.md`
- "json format" → `firecrawl-cli scrape <url> --json`
- "save json to data.json" → `firecrawl-cli scrape <url> --json > data.json`
- "only main content" → `firecrawl-cli scrape <url> --only-main`
- (no extra instructions) → `firecrawl-cli scrape <url>`

## Execution

1. Parse the URL and instructions from arguments
2. Build the appropriate `firecrawl-cli scrape` command
3. Run the command
4. If saving to file, confirm the file was created
5. If displaying output, show the scraped content
