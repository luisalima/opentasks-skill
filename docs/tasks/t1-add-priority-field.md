---
status: done
type: task
id: T1
deliverable: D1
created: 2026-06-11
links:
  - docs/adr/0001-agent-self-dispatch.md
started: 2026-06-11
closed: 2026-06-11
output: SKILL.md
---

# T1. Add `priority:` frontmatter field

## Objective
Give tasks an ordering signal so humans and agents can tell what matters most; today the index has no priority concept at all.

## What we need to extract / do
- Add optional `priority: p1|p2|p3` (default `p2` when absent) to the task frontmatter reference in `SKILL.md`.
- Show the priority as a tag on `TASK_INDEX.md` lines for `p1` and `p3` items (omit the default to keep lines quiet).
- Teach `sync` and `status` to validate the value (reject anything other than `p1`/`p2`/`p3`).
- Update `list` to accept a `p1`/`p2`/`p3` filter.
- Reflect the change in `README.md` per the canonicalization rules (see ADR-0002 / T9).

## Done when
- `SKILL.md` documents the field, its default, the index tag format, and the validation rule.
- `new task` asks for / infers priority only when context suggests it; otherwise omits the field.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
None.
