---
description: Review Convex backend for schema design, query patterns, performance, and best practices
argument-hint: <paths|staged|unstaged|all>
---

# Convex Review

Systematic review of Convex backend code for idiomatic patterns and best practices.

## What to Review

**Target:** $ARGUMENTS

If no target specified, ask user:

- `staged` — review staged changes only
- `unstaged` — review unstaged changes only
- `all` — review entire convex/ directory
- `<paths>` — review specific files or folders

## Review Process

### Step 1: Identify Scope

Determine what's being reviewed:

- Schema files (schema.ts)
- Function files (queries, mutations, actions)
- Both

### Step 2: Run Specialized Reviewers in Parallel

Use Task tool to run **both reviewers simultaneously**:

**1. Convex Architect** — Use **convex-architect** agent to analyze:

- Schema design (Table usage, field ordering, validators, indexes)
- File organization and naming conventions
- Cross-file consistency
- Documentation alignment

**2. Convex Patterns** — Use **convex-patterns** agent to analyze:

- DRY principle adherence (pick/omit/partial usage)
- Index utilization (range queries vs filter)
- Handler anti-patterns
- Return type consistency
- Performance concerns

### Step 3: Cross-Cutting Analysis

**ULTRA-THINK.** After reviewers complete, personally analyze for:

**Schema-Function Alignment:**

- Do function args match schema field order?
- Are all indexed fields utilized correctly in queries?
- Do return types accurately reflect schema + system fields?

**Performance Patterns:**

- Are indexes covering actual query patterns?
- Any missing indexes causing full table scans?
- Index field order matching query usage?

**Best Practices Adherence:**

- Does implementation follow documented patterns (README.md, docs)?
- Are there project-specific conventions being violated?
- Any deviations from standard Convex patterns?

## Final Report

**ULTRA-THINK before writing.** Synthesize findings into unified report:

```markdown
# Convex Review Summary

## Overview

[What was reviewed — files, scope, context]

## P1 — Critical Issues

[Schema violations, performance issues, broken patterns]
[If none: "None found"]

## P2 — Important Improvements

[DRY violations, missing indexes, inconsistent patterns]
[If none: "None found"]

## P3 — Minor Improvements

[Documentation, naming, minor optimizations]
[If none: "Code follows best practices"]

## Detailed Findings

### Schema & Structure

[Key findings from convex-architect agent]

### Implementation Patterns

[Key findings from convex-patterns agent]

### Performance Analysis

[Index usage, query efficiency, potential bottlenecks]

## Recommended Actions

[Prioritized list of specific improvements]
[If code is clean: "No significant improvements needed"]
```

## Priority Definitions

**P1 — Critical**

- Schema violations that break indexing
- O(n) queries where O(log n) is possible
- Missing DRY that causes schema drift
- Security issues (missing validation)

**P2 — Important**

- Inconsistent use of pick/omit/partial
- Field ordering violations
- Moderate complexity that could be simplified
- Missing performance optimizations

**P3 — Minor**

- Documentation improvements
- Naming consistency
- Small refactoring opportunities
