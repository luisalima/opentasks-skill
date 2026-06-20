---
status: todo
type: task
id: T11
deliverable: D1
created: 2026-06-20
links:
  - docs/feedback/multi-agent-automation-feedback.md
priority: p1
depends_on: [T12, T13]
---

# T11. Add `autonomy:` frontmatter field

## Objective
Give a driver a per-task signal of which tasks are safe to run unattended. Today nothing in the frontmatter distinguishes an agent-runnable task from one that needs a person, so an unattended driver cannot decide what to pick up. (Automation-pilot feedback #2.)

## What we need to extract / do
- Add optional `autonomy: auto | human` to the task frontmatter reference in `SKILL.md` and the README. Absent means `human` — the safe default.
- `autonomy: auto` — an agent/driver may execute this task unattended.
- `autonomy: human` — needs a person (open decision, too large, audit, or otherwise risky).
- Document the bar an `auto` task must clear: `status: todo`, every `depends_on` is `done` (readiness, T2), no linked open decision (`blocked_by`, T13), and a concrete machine-checkable `Done when` (T12).
- Teach `status` (and optionally `next`) to report the count of `auto`-eligible ready tasks.
- Validate the field in `sync`/`status`: values other than `auto`/`human` are mismatches.

## Done when
- `SKILL.md` and `README.md` document the field, its `human` default, and the `auto` eligibility bar.
- `status` reports the count of `auto`-eligible ready tasks.
- `sync`/`status` flag an invalid `autonomy` value as a frontmatter mismatch.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
The `auto` eligibility bar is defined in terms of an independently-verifiable `Done when` (T12) and a `blocked_by` link to any open decision (T13); both should land first so the bar references real fields. Builds on `depends_on` readiness (T2, done).
