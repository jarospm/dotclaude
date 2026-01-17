# firecrawl-cli

Custom CLI for the [Firecrawl](https://firecrawl.dev) API.

## Source

- GitHub: https://github.com/jarospm/firecrawl-cli
- Local: `~/tools/firecrawl-cli/`

## Used By

- `commands/firecrawl-search.md`
- `commands/firecrawl-scrape.md`
- `commands/firecrawl-agent.md`

## Prerequisites

- Node.js >= 20.0.0
- Firecrawl API key ([get one here](https://firecrawl.dev))

## Setup

```bash
cd ~/tools/firecrawl-cli
npm install
npm link  # makes `firecrawl-cli` available globally
```

## Configuration

Create `~/.config/firecrawl/.env`:

```bash
mkdir -p ~/.config/firecrawl
echo "FIRECRAWL_API_KEY=your_key_here" > ~/.config/firecrawl/.env
```

## Quick Reference

```bash
firecrawl-cli scrape <url>              # Extract content from URL
firecrawl-cli search "query"            # Search the web
firecrawl-cli search "query" --scrape   # Search and scrape results
firecrawl-cli map <url>                 # Discover URLs on a site
firecrawl-cli crawl <url>               # Recursively crawl
firecrawl-cli agent "prompt"            # AI-powered data gathering
```

Common flags: `--json`, `--quiet`, `--limit <n>`

## Full Documentation

See `~/tools/firecrawl-cli/README.md`
