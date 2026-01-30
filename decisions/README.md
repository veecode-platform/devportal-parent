# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records documenting significant decisions made in the VeeCode DevPortal project.

## What is an ADR?

An Architecture Decision Record captures an important architectural decision along with its context and consequences. ADRs are:

- **Immutable** — Once accepted, the record is not modified (superseded instead)
- **Numbered** — Sequential numbering for easy reference
- **Discoverable** — Easy to find and understand

## ADR Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| — | — | — | — |

*No ADRs have been recorded yet. Use the template below to create the first one.*

## Creating an ADR

### When to Create

Create an ADR when making decisions that:

- Affect the overall architecture
- Have significant implementation effort
- Are difficult to reverse
- Impact multiple repositories
- Change how the system operates

### Template

Create a new file: `NNNN-title-with-dashes.md`

```markdown
# NNNN. Title

## Status

Proposed | Accepted | Deprecated | Superseded by [NNNN](./NNNN-title.md)

## Date

YYYY-MM-DD

## Context

What is the issue that we're seeing that is motivating this decision or change?

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change?

### Positive

- Benefit 1
- Benefit 2

### Negative

- Drawback 1
- Drawback 2

### Neutral

- Side effect 1

## Alternatives Considered

### Alternative 1

Description and why it was rejected.

### Alternative 2

Description and why it was rejected.

## References

- [Link to discussion](https://...)
- [Link to documentation](https://...)
```

## Example ADRs to Record

When the time is right, consider documenting:

1. **Dynamic Plugin System Choice**
   - Why dynamic over static plugins?
   - How does it compare to RHDH approach?

2. **Profile-Based Configuration**
   - Why profiles over environment-only config?
   - Trade-offs of pre-defined profiles

3. **Multi-Repo vs Monorepo Structure**
   - Why separate repositories?
   - How do they coordinate?

4. **Base Image Selection (Red Hat UBI)**
   - Why UBI over Alpine or Debian?
   - Security and compatibility considerations

5. **Plugin Development in Separate Repo**
   - Why not develop plugins in devportal-base?
   - Benefits of workspace isolation

## ADR Lifecycle

```
┌──────────┐     ┌──────────┐     ┌────────────┐
│ Proposed │────▶│ Accepted │────▶│ Deprecated │
└──────────┘     └──────────┘     └────────────┘
                      │                  │
                      │                  │
                      ▼                  ▼
               ┌────────────┐     ┌─────────────┐
               │ Superseded │     │   Rejected  │
               └────────────┘     └─────────────┘
```

- **Proposed** — Under discussion
- **Accepted** — Decision made and implemented
- **Deprecated** — No longer relevant but kept for history
- **Superseded** — Replaced by a new decision
- **Rejected** — Proposed but not accepted

## References

- [ADR GitHub Organization](https://adr.github.io/)
- [Michael Nygard's Article](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Joel Parker Henderson's ADR Template](https://github.com/joelparkerhenderson/architecture-decision-record)
