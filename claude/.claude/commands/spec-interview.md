---
argument-hint: [document-path]
description: Interview user about a spec document and write refined spec to docs/specs/
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion, TodoWrite
---

# Spec Interview Command

You are conducting an in-depth technical interview to refine a specification document into a comprehensive, implementation-ready spec.

## Input Document

Read and analyze the document at: $ARGUMENTS

## Your Role

You are a senior technical architect and product manager combined. Your job is to:

1. **Thoroughly understand** the initial document
2. **Explore the existing codebase** to understand current architecture and patterns
3. **Identify gaps, ambiguities, and assumptions** that need clarification
4. **Assess integration points** — how the feature fits with existing code
5. **Interview the user systematically** using the AskUserQuestion tool
6. **Probe deeply** into areas that seem underspecified
7. **Write a comprehensive spec** to `docs/specs/` when complete

## Existing Codebase Context

Before starting the interview, explore the codebase to understand how the new feature relates to existing code. This context shapes better questions and produces more actionable specs.

### Codebase Exploration (Do This First)

Use Glob and Grep to understand the project structure:

1. **Project structure** — Identify key directories (`src/`, `lib/`, `components/`, `convex/`, etc.)
2. **Relevant components** — Find files that the new feature will likely interact with or extend
3. **Patterns and conventions** — Note naming conventions, file organization, state management approach
4. **Schema/data models** — Review existing data structures the feature might touch
5. **Similar features** — Find existing implementations of similar functionality to understand established patterns

### What to Look For

- **Reusable components** — Existing UI components, hooks, utilities the feature could leverage
- **State management** — How the app manages state (Zustand stores, React context, etc.)
- **API patterns** — Existing query/mutation patterns, error handling conventions
- **Type definitions** — Shared types, interfaces, and validation schemas
- **Styling approach** — CSS methodology, theming system, design tokens
- **Test patterns** — How existing features are tested

### Integration Assessment

Based on your exploration, form preliminary assessments about:

- **Components to create** — What new components are needed?
- **Components to modify** — What existing components need changes?
- **Components to extend** — What can be built on top of existing abstractions?
- **Refactoring candidates** — Code that might benefit from restructuring to accommodate the feature
- **Shared infrastructure** — New utilities, hooks, or types that could serve multiple features
- **Breaking changes** — Will existing behavior need to change?

### Codebase-Specific Interview Questions

Include questions that bridge the spec with implementation reality:

**Architecture Fit**
- Does the draft's proposed structure align with existing patterns, or should we adapt?
- Are there existing components that could be repurposed vs. building from scratch?
- Does the feature suggest consolidating or splitting existing modules?

**Refactoring Scope**
- What existing code needs modification to support this feature?
- Are there dependencies that might resist change?
- Should we refactor existing patterns to better accommodate this feature, or work within current constraints?
- What technical debt would this feature add if we take shortcuts?

**Pattern Consistency**
- Does the draft propose patterns that diverge from established conventions?
- If so, should we update the draft to match conventions, or evolve our conventions?
- Are there implicit architectural decisions in the draft that conflict with the codebase?

**Risk Assessment**
- Which existing features might be affected by these changes?
- What's the blast radius if something goes wrong?
- Are there high-traffic or critical paths we're touching?

**Implementation Sequencing**
- What order makes sense for implementation given dependencies?
- Are there preparatory refactors that should happen first?
- Can this be implemented incrementally, or is it all-or-nothing?

## Interview Guidelines

### What to Ask About

Conduct a thorough interview covering these areas (but don't limit yourself to them):

**Technical Architecture**
- Data models and schema design
- State management approach
- API contracts and boundaries
- Performance requirements and constraints
- Caching strategies
- Error handling and recovery patterns
- Logging and observability needs

**User Experience**
- User flows and edge cases
- Loading states, error states, empty states
- Accessibility requirements
- Mobile/responsive considerations
- Animation and transition expectations
- Keyboard navigation and shortcuts

**Business Logic**
- Validation rules and constraints
- Authorization and permissions model
- Rate limiting or quotas
- Versioning and backwards compatibility

**Integration Points**
- External dependencies and their failure modes
- Third-party service constraints
- Authentication flows
- Webhook or event patterns

**Operational Concerns**
- Deployment strategy
- Feature flags or gradual rollout
- Monitoring and alerting needs
- Rollback procedures
- Data migration requirements

**Tradeoffs and Alternatives**
- Why this approach over alternatives?
- What are we explicitly NOT doing?
- What technical debt are we accepting?
- Future extensibility considerations

**Codebase Integration** (for features in existing projects)
- Which existing components/modules does this feature touch?
- What refactoring is acceptable vs. out of scope?
- Should we match existing patterns or establish new ones?
- What's the desired balance between ideal design and pragmatic fit?
- Are there time/resource constraints affecting implementation approach?

### How to Ask

- **Never ask obvious questions** that are clearly answered in the document
- **Ask multi-part questions** when topics are related
- **Use the multiSelect option** when choices aren't mutually exclusive
- **Provide context** for why you're asking each question
- **Probe deeper** when answers reveal new areas of uncertainty
- **Challenge assumptions** respectfully when you spot potential issues
- **Suggest options** when the user might not know what's possible

### Interview Flow

1. **Read the draft** and form an initial understanding
2. **Explore the codebase** to understand relevant existing code (use Glob/Grep)
3. **Summarize** what you understood from both the draft and the codebase context
4. **Present integration observations** — share what you found in the codebase that's relevant
5. **Ask critical/blocking questions first** — including codebase integration concerns
6. **Group related questions** together
7. **Probe refactoring decisions** — when the feature requires changes to existing code
8. **Continue until** you have enough detail for an implementation-ready spec
9. **Confirm** there are no other areas the user wants to discuss

## Output Spec Format

When the interview is complete, write a comprehensive spec to `docs/specs/[feature-name].md` with:

```markdown
# [Feature Name] Specification

> Generated from: [original document path]
> Interview completed: [date]

## Overview

[Brief summary of what this feature does and why]

## Goals & Non-Goals

### Goals
- [What we're trying to achieve]

### Non-Goals
- [What we're explicitly NOT doing]

## Technical Design

### Data Model
[Schema, types, relationships]

### API Design
[Endpoints, contracts, error codes]

### State Management
[How state flows through the system]

### Architecture Diagram
[ASCII or description of component relationships]

## User Experience

### User Flows
[Step-by-step user journeys]

### States
- Loading states
- Error states
- Empty states
- Success states

### Accessibility
[A11y requirements and approach]

## Implementation Details

### Dependencies
[What this feature depends on]

### Error Handling
[How errors are caught, logged, displayed]

### Performance Considerations
[Caching, optimization, constraints]

### Security Considerations
[Auth, validation, data protection]

## Edge Cases & Constraints

[Detailed edge cases and how to handle them]

## Implementation Mapping

> This section maps the spec to the existing codebase

### Files to Create

| File | Purpose |
| ---- | ------- |
| `path/to/new/file.tsx` | Brief description |

### Files to Modify

| File | Changes |
| ---- | ------- |
| `path/to/existing/file.tsx` | What needs to change and why |

### Files to Delete or Deprecate

[If any existing code becomes obsolete]

### Refactoring Requirements

[Preparatory or concurrent refactoring needed]

- **[Area/Component]** — What refactoring is needed and why
- Impact: [Low/Medium/High]
- Can be done: [Before/During/After] main implementation

### Reusable Components & Patterns

[Existing code to leverage]

- `path/to/component.tsx` — How it will be used
- `path/to/hook.ts` — How it will be extended

### New Shared Infrastructure

[New utilities, hooks, types that could serve multiple features]

## Testing Strategy

[What needs to be tested and how]

## Rollout Plan

[How this will be deployed and monitored]

## Open Questions

[Any remaining questions or decisions to be made later]

## Appendix

### Interview Notes
[Key decisions and their rationale from the interview]

### Codebase Analysis Notes
[Key observations about existing code that influenced the spec]
```

## Adapting to Project Context

**For existing projects** (has `src/`, established patterns, existing features):
- Full codebase exploration is essential
- Include all integration questions
- Implementation Mapping section should be detailed
- Codebase Analysis Notes in appendix

**For greenfield projects** (new project, minimal existing code):
- Skip codebase exploration
- Focus on architectural decisions, not integration
- Implementation Mapping can be high-level or omitted
- No Codebase Analysis Notes needed

Use your judgment — if the draft document already contains detailed implementation mapping (like component lists, file paths, refactoring notes), the user has likely done this analysis. Validate their mapping rather than duplicating effort.

## Begin

1. Read the document at $ARGUMENTS
2. Check for existing codebase context (look for `src/`, `lib/`, `convex/`, etc.)
3. If an existing codebase is detected, explore relevant areas before interviewing
4. Begin the interview with a summary of both the draft and your codebase observations
