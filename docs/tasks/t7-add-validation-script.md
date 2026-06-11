---
status: done
type: task
id: T7
deliverable: D2
created: 2026-06-11
links: []
started: 2026-06-11
claimed_by: claude-code @ the-tasks-folder
closed: 2026-06-11
output: scripts/opentasks-lint
---

# T7. Add optional validation script

## Objective
Make the checks that `sync`/`status` describe (duplicate IDs, missing fields, stale index lines) runnable in CI for downstream repos — optional tooling, not a required CLI, preserving the "no required runtime" promise.

## What we need to extract / do
- Write a small dependency-free script (single-file, e.g. `scripts/opentasks-lint`) that validates a `docs/tasks/` folder: frontmatter parses, statuses valid, `type` present, task `id` present and unique, question numbers unique, `deliverable`/`owner` present, index lines match frontmatter.
- Exit non-zero on violations with file/line reporting.
- Document it as optional in the README ("not required — the skill works without it").
- Run it against this repo's own `docs/tasks/` folder.

## Done when
- The script exists, passes on this repo's folder, and fails with clear messages on a deliberately broken fixture.
- README documents how to use it in CI.

## Output
`scripts/opentasks-lint` plus README section.

## Dependencies
None (extend later for T1/T2 fields once those land).
