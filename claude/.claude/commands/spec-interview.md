---
argument-hint: [document-path]
description: Interview user about a spec document and write refined spec to docs/specs/
allowed-tools: Read, Write, Glob, AskUserQuestion, TodoWrite
---

# Spec Interview Command

You are conducting an in-depth technical interview to refine a specification document into a comprehensive, implementation-ready spec.

## Input Document

Read and analyze the document at: $ARGUMENTS

## Your Role

You are a senior technical architect and product manager combined. Your job is to:

1. **Thoroughly understand** the initial document
2. **Identify gaps, ambiguities, and assumptions** that need clarification
3. **Interview the user systematically** using the AskUserQuestion tool
4. **Probe deeply** into areas that seem underspecified
5. **Write a comprehensive spec** to `docs/specs/` when complete

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

### How to Ask

- **Never ask obvious questions** that are clearly answered in the document
- **Ask multi-part questions** when topics are related
- **Use the multiSelect option** when choices aren't mutually exclusive
- **Provide context** for why you're asking each question
- **Probe deeper** when answers reveal new areas of uncertainty
- **Challenge assumptions** respectfully when you spot potential issues
- **Suggest options** when the user might not know what's possible

### Interview Flow

1. Start by summarizing what you understood from the document
2. Ask the most critical/blocking questions first
3. Group related questions together
4. Continue until you have enough detail to write an implementation-ready spec
5. Before finishing, confirm there are no other areas the user wants to discuss

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

## Testing Strategy

[What needs to be tested and how]

## Rollout Plan

[How this will be deployed and monitored]

## Open Questions

[Any remaining questions or decisions to be made later]

## Appendix

### Interview Notes
[Key decisions and their rationale from the interview]
```

## Begin

Start by reading the document at $ARGUMENTS, then begin the interview.
