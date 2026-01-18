---
description: Run comprehensive code review with simplicity and TypeScript quality checks
argument-hint: <paths|staged|unstaged|PR-url>
---

# Code Review

Run a comprehensive code review using specialized reviewers.

## What to Review

**Target:** $ARGUMENTS

If no target is specified, ask the user what they want reviewed:
- `staged` — review staged changes only
- `unstaged` — review unstaged changes only
- `<paths>` — review specific files or folders
- `<PR URL>` — review a pull request

## Review Process

### Step 1: Gather Code

- For `staged`: run `git diff --cached`
- For `unstaged`: run `git diff`
- For paths: read files directly, or glob folders for source files
- For PR URLs: use `gh pr diff <number>` to get the changes

### Step 2: Run Reviewers in Parallel

Use the Task tool to run **both reviewers simultaneously**:

**1. Code Simplifier** — Use the **code-simplifier** agent to analyze:
- YAGNI violations and unnecessary complexity
- Code that can be removed or simplified
- Premature abstractions and over-engineering

**2. TypeScript Reviewer** (skip if no .ts/.tsx files) — Use the **typescript-reviewer** agent to analyze:
- Type safety issues and `any` usage
- Modern TypeScript patterns
- Naming clarity and testability

### Step 3: Cross-Cutting Analysis

**ULTRA-THINK.** Spend maximum cognitive effort on this synthesis. Think step by step. Consider all angles. Question assumptions. Cross-reference findings from both reviewers. Look for patterns they might have missed individually.

After reviewers complete, personally analyze for:

**Breaking Changes & Regressions:**
- Any deleted functionality that wasn't moved elsewhere?
- Changes to public APIs or interfaces?
- Modified behavior that existing code depends on?

**Edge Cases & Failure Scenarios:**
- Invalid/null/empty inputs handled?
- Boundary conditions (min/max values)?
- Concurrent access or race conditions?
- Network failures or timeouts?
- Error messages helpful to users?

**Developer Experience:**
- Is this easy to understand and modify?
- Can it be tested easily?
- Are high-level architectural comments present where needed?
  - Module purpose and responsibilities
  - Non-obvious design decisions (the "why")
  - How components interconnect
  - NOT line-by-line explanations of what code does

## Final Report

**ULTRA-THINK before writing.** Deeply consider: What are the most impactful improvements? What's the root cause behind multiple findings? What refactoring would address several issues at once? Be exhaustive but organized — capture all findings, deduplicate overlapping issues from both reviewers, group by root cause where applicable, and prioritize by impact.

Synthesize all findings into a unified report:

```markdown
# Code Review Summary

## Overview
[What was reviewed — files, PR title, scope]

## P1 — High Impact Improvements
[Security, major complexity, architectural issues, hard-to-maintain code]
[If none: "None found"]

## P2 — Medium Impact Improvements
[Performance, type safety, moderate complexity, error handling]
[If none: "None found"]

## P3 — Low Impact Improvements
[Minor cleanup, small optimizations, style consistency]
[If none: "Code looks good"]

## Detailed Findings

### Simplicity Analysis
[Key findings from code-simplifier agent]

### TypeScript Quality
[Key findings from typescript-reviewer agent, or "N/A — no TypeScript files"]

### Edge Cases & Risks
[Any concerns about failure scenarios, breaking changes, or regressions]

## Recommended Actions
[Prioritized list of specific improvements to make]
[If code is already clean: "No significant improvements needed"]
```

## Priority Definitions

**P1 — High Impact**
- Security vulnerabilities or data risks
- Significant complexity that hurts maintainability
- Major architectural issues
- Hard-to-understand or hard-to-test code

**P2 — Medium Impact**
- Performance improvements
- Type safety issues (`any` without justification)
- Moderate complexity that could be simplified
- Missing error handling

**P3 — Low Impact**
- Minor code cleanup
- Small optimization opportunities
- Style consistency
- Documentation additions
