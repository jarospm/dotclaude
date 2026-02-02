---
name: firecrawl
description: |
  Scrape, search, map, and crawl the web using Firecrawl CLI.

  Use this skill when:
  - You need FULL page content (not just snippets)
  - Standard WebSearch doesn't return useful results
  - You need to scrape JavaScript-rendered pages
  - You want to discover all URLs on a site (map)
  - You want to crawl/scrape an entire site or docs
  - You want to extract links from a page
  - User explicitly asks for firecrawl/scraping

  Do NOT use for:
  - Quick lookups — use WebSearch instead
  - Simple page fetches — use WebFetch instead
  - General research — start with WebSearch first
---

# Firecrawl CLI

Use the `firecrawl` CLI for full-content scraping and advanced web search.

**This skill complements WebFetch/WebSearch — it does not replace them.**

## When to Use Firecrawl

- **Full page content** — WebFetch returns summaries; firecrawl returns everything
- **JavaScript-heavy sites** — SPAs, dynamic content that WebFetch can't render
- **WebSearch failures** — when standard search doesn't find what you need
- **Site mapping** — discovering all URLs on a domain
- **Bulk scraping** — multiple pages in parallel
- **User explicitly requests** — "scrape this", "use firecrawl"

## Commands

### Scrape — extract full page content

```bash
# Basic scrape (markdown output)
firecrawl scrape <url>

# Clean content only (removes nav, footer, ads)
firecrawl scrape <url> --only-main-content

# Wait for JavaScript to render
firecrawl scrape <url> --wait-for 3000

# Save to file (recommended for large pages)
firecrawl scrape <url> -o output.md

# Multiple formats (returns JSON)
firecrawl scrape <url> --format markdown,links

# Extract links only
firecrawl scrape <url> --format links
```

**Key flags:**

- `--only-main-content` — removes navigation, footers, ads
- `--wait-for <ms>` — wait for JS rendering
- `--format <types>` — markdown, html, links, screenshot
- `--include-tags <tags>` — only include specific elements
- `--exclude-tags <tags>` — exclude specific elements
- `-o <file>` — save output to file

### Search — web search with optional scraping

```bash
# Basic search
firecrawl search "query"

# Limit results
firecrawl search "query" --limit 10

# Search AND scrape full content from results
firecrawl search "query" --scrape

# Filter by source
firecrawl search "query" --sources news
firecrawl search "query" --sources images

# Filter by category
firecrawl search "query" --categories github
firecrawl search "query" --categories research

# Time-based filtering
firecrawl search "query" --tbs qdr:d    # Past day
firecrawl search "query" --tbs qdr:w    # Past week

# Save results
firecrawl search "query" -o results.json --json
```

**Key flags:**

- `--limit <n>` — max results (default 5, max 100)
- `--scrape` — also scrape content from each result
- `--sources <list>` — web, news, images
- `--categories <list>` — github, research, pdf
- `--tbs <value>` — time filter (qdr:h, qdr:d, qdr:w, qdr:m, qdr:y)
- `--json` — output as JSON

### Map — discover URLs on a site

```bash
# List all URLs
firecrawl map <url>

# Filter by keyword
firecrawl map <url> --search "blog"

# Limit results
firecrawl map <url> --limit 100

# JSON output
firecrawl map <url> --json
```

### Crawl — scrape entire site recursively

Use crawl when you need content from multiple pages (e.g., entire docs site).

```bash
# Basic crawl (returns job ID immediately)
firecrawl crawl <url>

# Wait for completion with progress
firecrawl crawl <url> --wait --progress

# Limit pages and depth
firecrawl crawl <url> --limit 50 --max-depth 3

# Filter paths
firecrawl crawl <url> --include-paths "/docs/*"
firecrawl crawl <url> --exclude-paths "/admin/*,/api/*"

# Rate limiting
firecrawl crawl <url> --delay 1000 --max-concurrency 5

# Save output
firecrawl crawl <url> --wait -o site-content.json
```

**Key flags:**

- `--wait` — block until crawl completes (otherwise returns job ID)
- `--progress` — show progress while waiting
- `--limit <n>` — max pages to crawl
- `--max-depth <n>` — max link depth
- `--include-paths <paths>` — only crawl matching paths
- `--exclude-paths <paths>` — skip matching paths
- `--delay <ms>` — delay between requests
- `--allow-subdomains` — include subdomains

## Tips

**Always quote URLs** — shell interprets `?` and `&` as special characters:

```bash
firecrawl scrape "https://example.com/page?id=123&ref=home"
```

**Use `.firecrawl/` folder** for organizing outputs in a project:

```bash
mkdir -p .firecrawl
firecrawl scrape <url> -o .firecrawl/page.md
firecrawl crawl <url> --wait -o .firecrawl/docs.json
```

## Output Handling

For large outputs, save to file and read incrementally:

```bash
firecrawl scrape <url> -o .firecrawl/page.md
head -100 .firecrawl/page.md
grep -n "keyword" .firecrawl/page.md
```

**Processing JSON with jq:**

```bash
# Extract URLs from search results
firecrawl search "query" --json | jq -r '.data.web[].url'

# Get titles and URLs
firecrawl search "query" --json | jq -r '.data.web[] | "\(.title): \(.url)"'

# Extract links from scrape
firecrawl scrape <url> --format links | jq -r '.links[].url'

# Count URLs from map
firecrawl map <url> --json | jq '.urls | length'
```

## Parallelization

For multiple URLs, run in parallel:

```bash
firecrawl scrape https://site1.com -o 1.md &
firecrawl scrape https://site2.com -o 2.md &
firecrawl scrape https://site3.com -o 3.md &
wait
```

## Decision Guide

| Need                  | Tool                        |
| --------------------- | --------------------------- |
| Quick fact lookup     | WebSearch                   |
| Read a simple page    | WebFetch                    |
| Full page content     | firecrawl scrape            |
| JS-rendered content   | firecrawl scrape --wait-for |
| Search + full content | firecrawl search --scrape   |
| Find all URLs on site | firecrawl map               |
| Scrape entire site    | firecrawl crawl             |

## Troubleshooting

If something isn't working as expected:

```bash
firecrawl --help          # Available commands and flags
firecrawl --status        # Auth status, credits, concurrency
firecrawl login --browser # Re-authenticate if needed
```
