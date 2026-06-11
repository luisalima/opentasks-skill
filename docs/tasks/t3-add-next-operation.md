---
status: todo
type: task
id: T3
deliverable: D1
created: 2026-06-11
links:
  - docs/adr/0001-agent-self-dispatch.md
depends_on: [T1, T2]
---

# T3. Add `next` operation

## Objective
Answer "what should I pick up?" deterministically, so an agent can dispatch itself from the folder instead of being hand-fed a task.

## What we need to extract / do
- Add a `/opentasks next` operation to `SKILL.md`: among ready tasks (`todo` with all `depends_on` done), pick the highest priority; break ties by lowest task number.
- Output the chosen task's ID, title, file, and why it was chosen (priority, satisfied dependencies); list the runners-up briefly.
- If nothing is ready, say so and show what is blocking the highest-priority non-ready task.
- Optionally accept a deliverable filter: `next D1`.

## Done when
- `SKILL.md` documents the operation, including the tie-break rule and the empty-result behavior.
- The README usage table includes `next`.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
T1 (priority field), T2 (depends_on field).
