---
status: done
type: task
id: T15
deliverable: D3
created: 2026-06-20
links:
  - docs/feedback/multi-agent-automation-feedback.md
priority: p3
started: 2026-06-23
claimed_by: claude-code @ claude-the-tasks-folder
closed: 2026-06-23
output: SKILL.md
---

# T15. Document recording provenance for review-spawned tasks

## Objective
When a task is spawned from a review or a finding, its origin (report path, PR, or finding id) should be traceable. The `links:` field already supports this; it just is not a documented habit. (Automation-pilot feedback #6 — and this very task records its own source in `links:`.)

## What we need to extract / do
- Add a short convention to the "When agents create tasks" guidance in `SKILL.md` and the README: when a task is spawned from a review or finding, record the source (report path, PR, or finding id) in `links:`.
- No new field — this documents an existing capability.

## Done when
- `SKILL.md` and `README.md` state that review/finding-spawned tasks record their source in `links:`.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
None.
