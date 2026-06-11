---
status: done
type: task
id: T2
deliverable: D1
created: 2026-06-11
links:
  - docs/adr/0001-agent-self-dispatch.md
started: 2026-06-11
closed: 2026-06-11
output: SKILL.md
---

# T2. Add `depends_on:` frontmatter field

## Objective
Make task dependencies machine-readable. The `## Dependencies` body section is free text, so nothing can compute readiness, detect cycles, or draw a graph.

## What we need to extract / do
- Add optional `depends_on: [T<N>, …]` (YAML list of task IDs) to the task frontmatter reference in `SKILL.md`.
- Define "ready": `status: todo` and every `depends_on` entry is `done`.
- Teach `sync` and `status` to validate: unknown task IDs, self-references, and cycles are reported as mismatches.
- Teach `status` to flag `doing` tasks whose dependencies are not `done`.
- Keep the `## Dependencies` body section for prose context; `depends_on` is the normative list.

## Done when
- `SKILL.md` documents the field, the readiness rule, and the validations.
- `status` output includes a "not ready / dependency violation" report when applicable.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
None.
