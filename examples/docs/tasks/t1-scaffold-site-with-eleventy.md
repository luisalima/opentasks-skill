---
status: done
type: task
id: T1
deliverable: D1
created: 2026-05-20
links:
  - docs/adr/0001-use-eleventy.md
started: 2026-05-21
claimed_by: claude-code @ herbarium-main
closed: 2026-05-22
output: site/
---

# T1. Scaffold site with Eleventy

## Objective
Stand up the static site skeleton so content work can start. Generator choice was settled in ADR-0001.

## What we need to extract / do
- Initialize the Eleventy project under `site/` with the base layout.
- Add the plant content collection and a sample entry.
- Wire up `npm run dev` and `npm run build`.

## Done when
- `npm run build` produces the site with the sample plant page.

## Output
`site/` — the project skeleton everything else builds on.

## Dependencies
None.
