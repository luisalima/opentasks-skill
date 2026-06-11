---
status: todo
type: task
id: T4
deliverable: D1
created: 2026-06-04
links: []
priority: p1
depends_on: [T2]
---

# T4. Add client-side search

## Objective
Visitors mostly arrive looking for one specific plant; search is the fastest path there.

## What we need to extract / do
- Build a search index at build time (title, latin name, tags).
- Add a search box to the index page with client-side matching.
- No external service — static JSON plus a small script.

## Done when
- Typing a plant name on the index page surfaces it within the first three results.

## Output
`site/search.json` build step and the search component.

## Dependencies
T2 — the index page hosts the search box.
