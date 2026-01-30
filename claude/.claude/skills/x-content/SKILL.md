---
name: x-content
description: Fetch content from X/Twitter using bird CLI. Use when the user shares x.com or twitter.com URLs, or asks about their X bookmarks, likes, mentions, timeline, or tweets.
---

# X/Twitter Content Skill

Use `bird` CLI to fetch content from X (formerly Twitter).

## When to Use

- User shares an x.com or twitter.com URL
- User asks about their X bookmarks, likes, mentions, or timeline
- User wants to search tweets or read a thread
- User asks about content from specific X accounts

## Quick Reference

```bash
# Read content
bird read <tweet-url-or-id>      # Fetch single tweet
bird thread <id>                 # Full conversation thread
bird replies <id>                # Replies to a tweet
bird user-tweets @handle         # User's timeline

# User's own content
bird bookmarks                   # Bookmarked tweets (default: 20)
bird bookmarks -n 50             # More bookmarks
bird bookmarks --all             # All bookmarks (paginated)
bird likes                       # Liked tweets
bird mentions                    # Tweets mentioning you

# Search
bird search "query"              # Search tweets
bird search "from:@handle topic" # Search specific user

# Social
bird following                   # Users you follow
bird followers                   # Your followers

# News/Trending
bird news                        # AI-curated news from Explore
bird news --ai-only -n 20        # Only AI-curated items
```

## Output Flags

- `--json` — structured JSON output
- `--plain` — no emoji, no color (script-friendly)
- `-n <count>` — limit number of results

## Examples

**User shares a tweet URL:**
```bash
bird read https://x.com/someone/status/1234567890
```

**User asks "show my recent bookmarks about AI":**
```bash
bird bookmarks -n 50 --json
# Then filter/analyze the JSON output for AI-related content
```

**User asks "what's @elonmusk been posting?":**
```bash
bird user-tweets @elonmusk -n 20
```

**User asks "find tweets about Claude from my likes":**
```bash
bird likes --json
# Then filter for "Claude" mentions
```

## Tips

- For filtering content (e.g., "bookmarks about X topic"), fetch with `--json`
  and analyze the output
- Tweet IDs can be extracted from URLs — bird accepts both formats
- Use `bird whoami` to verify authentication if commands fail
