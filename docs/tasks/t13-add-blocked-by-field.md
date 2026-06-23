---
status: done
type: task
id: T13
deliverable: D1
created: 2026-06-20
links:
  - docs/feedback/multi-agent-automation-feedback.md
started: 2026-06-23
claimed_by: claude-code @ claude-the-tasks-folder
closed: 2026-06-23
output: SKILL.md
---

# T13. Add optional `blocked_by:` to tie a human task to its open decision

## Objective
A task whose blocker is an unresolved decision should point at the question that gates it, so the automation gate and the `Q → ADR → T` flow stay consistent. Today that link is informal — it lives in prose or freeform `links:`. (Automation-pilot feedback #7.)

## What we need to extract / do
- Add an optional `blocked_by: Q<N>` frontmatter field (a question id) to the reference in `SKILL.md` and the README.
- Document the rule: a task whose blocker is an open decision references the question in `blocked_by` and stays `autonomy: human` until that question closes (ties into T11).
- Teach `sync`/`status` to validate `blocked_by`: an unknown question id is a mismatch; optionally flag a task that is `autonomy: auto` while `blocked_by` an unanswered question.
- Keep `links:` for general provenance; `blocked_by` is the normative decision link.

## Done when
- `SKILL.md` and `README.md` document `blocked_by`, its relationship to `autonomy`, and the validation.
- `sync`/`status` flag an unknown `blocked_by` reference as a frontmatter mismatch.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
None structurally, but pairs with `autonomy` (T11): a `blocked_by` open question keeps a task `human`.
