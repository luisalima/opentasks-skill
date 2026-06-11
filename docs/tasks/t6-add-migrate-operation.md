---
status: done
type: task
id: T6
deliverable: D2
created: 2026-06-11
links:
  - docs/adr/0003-convention-migration.md
started: 2026-06-11
claimed_by: claude-code @ the-tasks-folder
closed: 2026-06-11
output: SKILL.md
---

# T6. Add `migrate` operation

## Objective
Give existing `docs/tasks/` folders a way to catch up when the convention changes. `sync` detects legacy files but nothing fixes them.

## What we need to extract / do
- Add a `/opentasks migrate` operation to `SKILL.md` that upgrades legacy frontmatter in place: add missing `type`, assign `id: T<N>` (from `t<N>-` filenames where present, otherwise allocate the next free number), add missing `links: []`.
- Make it idempotent — running on an up-to-date folder changes nothing.
- After rewriting files, run `sync` and report every change made, file by file.
- Add a CONTRIBUTING rule: any convention change affecting existing files must extend `migrate` in the same PR.

## Done when
- `SKILL.md` documents the operation and its idempotency guarantee.
- CONTRIBUTING's "describe migration impact" checklist item points at extending `migrate`.

## Output
Updated `SKILL.md`, `README.md`, `CONTRIBUTING.md`.

## Dependencies
None.
