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
- User wants to follow/unfollow accounts or post tweets

## Commands

### Reading Content

```bash
bird <url-or-id>                 # Read single tweet (shorthand)
bird read <url-or-id>            # Same as above
bird thread <id>                 # Full conversation thread
bird replies <id>                # Replies to a tweet
bird user-tweets @handle -n 20   # User's profile timeline
bird about @handle               # Account info (origin, location)
```

### Timelines

```bash
bird home                        # Home timeline (For You)
bird home --following            # Following-only timeline
bird mentions                    # Tweets mentioning you
bird mentions --user @handle     # Mentions of another user
```

### Bookmarks & Likes

```bash
bird bookmarks -n 20             # Recent bookmarks
bird bookmarks --all             # All bookmarks (paginated)
bird bookmarks --include-parent  # Include parent tweet context
bird bookmarks --author-chain    # Author's self-reply chain
bird likes -n 20                 # Liked tweets
bird unbookmark <id>             # Remove bookmark
```

### Search

```bash
bird search "query" -n 10
bird search "from:handle topic"  # Search specific user
bird search "query" --all --max-pages 3
```

### Lists

```bash
bird lists                       # Your lists
bird lists --member-of           # Lists you're a member of
bird list-timeline <id> -n 20    # Tweets from a list
```

### Social

```bash
bird following -n 20             # Users you follow
bird followers -n 20             # Your followers
bird follow @handle              # Follow a user
bird unfollow @handle            # Unfollow a user
```

### News & Trending

```bash
bird news -n 10                  # AI-curated from Explore
bird news --ai-only              # Only AI-curated items
bird trending                    # Alias for news
```

### Posting

```bash
bird tweet "hello world"
bird reply <id> "nice thread!"
bird tweet "check this" --media image.png --alt "description"
```

**Warning:** Posting is more likely to trigger rate limits. If blocked, suggest using the browser instead.

## Output Flags

- `-n <count>` — limit number of results
- `--json` — structured JSON output
- `--plain` — no emoji, no color (script-friendly)
- `--all` — fetch all pages (use with `--max-pages` to limit)

## Examples

**User shares a tweet URL:**
```bash
bird https://x.com/someone/status/1234567890
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

**User asks "what's on my timeline?":**
```bash
bird home --following -n 20
```

**User asks "find tweets about Claude from my likes":**
```bash
bird likes --json
# Then filter for "Claude" mentions
```

## Tips

- For filtering content (e.g., "bookmarks about X topic"), fetch with `--json` and analyze the output
- Tweet IDs can be extracted from URLs — bird accepts both formats
- Use `bird whoami` to verify authentication if commands fail

## Troubleshooting

```bash
bird whoami              # Check logged-in account
bird check               # Show credential sources
bird query-ids --fresh   # Fix 404 errors from stale query IDs
```

If cookie extraction fails:
- Ensure browser is logged into X
- Check `bird check` output for active cookie source
- For Arc/Brave: may need `--chrome-profile-dir` flag
