---
status: todo
type: task
id: T8
deliverable: D3
created: 2026-06-11
links: []
---

# T8. Add `examples/` folder with a populated sample

## Objective
Show a realistic populated `docs/tasks/` folder so humans evaluating the project and agents pattern-matching the convention can see it in use, not just read templates.

## What we need to extract / do
- Create `examples/docs/tasks/` with a small fictional project: 4–6 tasks across two deliverables covering all four statuses, 2 questions (one answered), a matching `TASK_INDEX.md`, and one linked ADR.
- Include at least one `done` task with an `output:` and one `blocked` task with a `## Blocker`.
- Link the example from the README.
- Keep it consistent with whatever fields exist when it lands (update if T1/T2/T4 have merged).

## Done when
- The example folder validates cleanly (by inspection, or via T7's script if available).
- README links to it.

## Output
`examples/docs/tasks/` and a README pointer.

## Dependencies
None.
