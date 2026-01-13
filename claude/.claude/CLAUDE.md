## Output formatting (IMPORTANT)

### Em-dash spacing

Always use spaces around em-dashes: ` — ` not `—`.

- Good: "This is important — never skip it"
- Bad: "This is important—never skip it"

This applies to ALL output — prose, code comments, commit messages, everything.

### Line length for readability

- Keep prose lines short (under 80-90 characters when practical)
- Prefer shorter sentences over long compound ones
- Use bullet points to break up dense explanations
- Avoid long inline code examples within paragraphs

### Markdown tables readability

NEVER use markdown tables when ANY column contains sentences or descriptions.
Tables break in narrow terminals and become unreadable.

Use tables ONLY for short-value comparisons like status grids (✅/❌, yes/no, numbers).

For "Feature → Impact" or "Thing → Description" patterns, ALWAYS use a list:

- **Feature name** – description here
- **Another feature** – another description

## Insight box formatting (Explanatory mode)

When using Explanatory output style, format insights as SHORT bullet points:

**★ Insight**
- First point here (under 80-90 characters per bullet)
- Second point here
- Keep each bullet concise to avoid word-wrap bugs

Rules:
- Use bullets, not prose paragraphs
- Each bullet must be under 90 characters
- No blockquotes, backticks, or box-drawing characters (───)
- Dense paragraphs break word wrapping in Claude Code terminals
