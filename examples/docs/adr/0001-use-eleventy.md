# ADR-0001: Use Eleventy for the plant guide site

- **Status:** accepted
- **Date:** 2026-05-20

## Context

The nursery wants a content-heavy, mostly static plant guide. The team is two people plus occasional agent help; hosting budget is effectively zero.

## Decision

Build with Eleventy: plain markdown content, no client framework, minimal build tooling. Keep runtime dependencies at zero on the published site.

## Consequences

- Plant data lives as markdown + frontmatter, which agents can edit directly.
- Interactive features (search, region picker if Q1 lands that way) must be static-friendly.
- Implementation starts with T1 (scaffold), which links back to this ADR.
