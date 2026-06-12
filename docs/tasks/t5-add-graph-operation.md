---
status: done
started: 2026-06-12
closed: 2026-06-12
claimed_by: claude-code @ Mac
output: SKILL.md
type: task
id: T5
deliverable: D1
created: 2026-06-11
links:
  - docs/adr/0001-agent-self-dispatch.md
depends_on: [T2]
---

# T5. Add `graph` operation (Mermaid)

## Objective
Visualize the task dependency graph from `depends_on`, so structure that currently lives in people's heads shows up in the repo. GitHub renders Mermaid natively.

## What we need to extract / do
- Add a `/opentasks graph` operation to `SKILL.md`: read `depends_on` across all task files and emit a Mermaid `graph` (flowchart) block.
- Style nodes by status (e.g. done greyed, doing highlighted, blocked marked).
- Decide and document where the output goes: printed to the user by default, optionally written into a fenced block in `TASK_INDEX.md` that `sync` regenerates.
- Handle the empty case (no `depends_on` anywhere) with a clear message instead of an empty diagram.

## Done when
- `SKILL.md` documents the operation, node styling, and output destination.
- The README usage table includes `graph`.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
T2 (depends_on field).
