# Firecrawl CLI

Official CLI for [Firecrawl](https://firecrawl.dev) — turn websites into LLM-ready data.

## Source

- Docs: https://docs.firecrawl.dev/sdks/cli
- GitHub: https://github.com/firecrawl/cli
- npm: `firecrawl-cli` (command is `firecrawl`)

## Installation

```bash
npm install -g firecrawl-cli
firecrawl login --browser
```

## Core Commands

### Scrape — extract content from a URL

```bash
firecrawl scrape <url>
firecrawl scrape <url> --only-main-content    # Clean output (recommended)
firecrawl scrape <url> --format markdown,links
firecrawl scrape <url> --wait-for "selector"  # Wait for JS to render
firecrawl scrape <url> -o output.md           # Save to file
```

### Search — query the web

```bash
firecrawl search "query"
firecrawl search "query" --limit 10
firecrawl search "query" --scrape             # Also scrape results
firecrawl search "query" --source news        # web | news | images
firecrawl search "query" --location "Berlin,Germany"
```

### Map — discover URLs on a site

```bash
firecrawl map <url>
firecrawl map <url> --search "blog"           # Filter by keyword
firecrawl map <url> --limit 100
firecrawl map <url> --sitemap only            # Use sitemap exclusively
firecrawl map <url> --ignore-query-parameters # Deduplicate URLs
```

### Crawl — recursive site crawling

```bash
firecrawl crawl <url> --wait --progress       # Block until done
firecrawl crawl <url> --limit 50
firecrawl crawl <url> --max-depth 3
firecrawl crawl <url> --include-paths "/blog/*"
firecrawl crawl <url> --exclude-paths "/admin/*"
firecrawl crawl <url> --delay 1000            # Rate limit (ms)
```

## Key Flags

- `--only-main-content` — removes nav, footers, ads (cleaner for LLMs)
- `--format <types>` — markdown, html, rawHtml, links, screenshot, json
- `--wait` — block until crawl job completes
- `--progress` — show visual progress bar
- `--pretty` — pretty-print JSON output
- `-o <file>` — save output to file

## Piping & Integration

```bash
# Extract URLs with jq
firecrawl map https://example.com --format json | jq -r '.urls[]'

# Scrape and process
firecrawl scrape https://example.com | pbcopy
```

## Configuration

Credentials stored after `firecrawl login`. To use env var instead:

```bash
export FIRECRAWL_API_KEY=fc-your-key
```

Disable telemetry:

```bash
export FIRECRAWL_NO_TELEMETRY=1
```

## Agent Skill

Install as Claude Code skill for native integration:

```bash
npx skills add firecrawl/cli -a claude-code -g
```

See `tools/ecosystems/vercel-skills-cli.md` for skill management details.
