---
name: assemble-board
description: Assemble a board of experts for a project. Analyzes the project and creates specialized subagents that can be consulted for reviews and perspectives. Invoked with /assemble-board.
disable-model-invocation: true
---

# Assemble Board of Experts

Create a tailored "board of experts" for this project — specialized subagents that can be consulted for reviews, advice, and domain-specific perspectives.

## Process

1. **Understand the project** — read config files, scan structure, identify the domain
2. **Determine what expertise would be valuable** — think about what perspectives would catch blind spots, improve quality, or bring domain knowledge
3. **Propose the board** — present recommended roles for user approval
4. **Create each expert** as a subagent in `.claude/agents/<role-name>.md`

## Subagent Structure

Each expert subagent must include this frontmatter:

```yaml
---
name: <role-name>
description: <Clear trigger — when should Claude consult this expert?>
tools: Read, Glob, Grep
model: inherit
---
```

The markdown body defines the expert's persona — who they are, what they care about, how they think. Keep it focused on identity and perspective, not task instructions.

Claude will craft contextual task messages when consulting experts.

## Key Principles

- Each expert should have a distinct, valuable perspective
- Descriptions should help Claude know when to consult each expert
- Personas should be opinionated — bland advice is useless
- Let the project's actual needs drive role selection
