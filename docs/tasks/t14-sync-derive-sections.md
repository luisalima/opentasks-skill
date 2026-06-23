---
status: todo
type: task
id: T14
deliverable: D2
created: 2026-06-20
links:
  - docs/feedback/multi-agent-automation-feedback.md
depends_on: [T11]
---

# T14. Teach `sync` to derive curated sections instead of clobbering them

## Objective
We want the index to show the auto/human split (and execution ordering) without hand-maintained sections that `sync` wipes on every rebuild. Prefer derivation over preservation. (Automation-pilot feedback #5.)

## What we need to extract / do
- Teach `sync` to derive an auto/human (autonomy) view from frontmatter, the same way it already regenerates the `## Dependency graph` section from `depends_on`.
- Confirm execution ordering is derived from `depends_on` (topological) and survives index rebuilds — already true for the dependency graph; extend the pattern if an explicit ordering view is wanted.
- If any genuinely manual prose must remain in `TASK_INDEX.md`, define a single clearly-delimited region that `sync` preserves; otherwise keep the index fully derived.
- Document what `sync` derives vs. preserves in the `sync` operation and §INDEX.

## Done when
- `sync` regenerates the autonomy split (and any ordering view) deterministically from frontmatter; no manual section is required for either.
- `SKILL.md` documents what `sync` derives vs. preserves.

## Output
Updated `SKILL.md`.

## Dependencies
Needs the `autonomy` field (T11) to exist before the split can be derived. Builds on the existing `depends_on`/`graph` derivation (T2, T5, done).
