---
status: done
type: task
id: T4
deliverable: D1
created: 2026-06-11
links:
  - docs/adr/0001-agent-self-dispatch.md
started: 2026-06-11
closed: 2026-06-11
output: SKILL.md
---

# T4. Add `claimed_by:` attribution to `start`

## Objective
Record who or what picked up a task and where it is running, and have agents announce the claim — attribution without locking, per ADR-0001.

## What we need to extract / do
- Extend the `start` operation in `SKILL.md`: write `claimed_by: <agent-or-person> @ <host-or-checkout>` to frontmatter alongside `started:`.
- Instruct the agent to announce the claim in its reply when starting a task (e.g. "Claimed T4 as claude-code on lu-macbook").
- Define what happens on re-`start` of an already-`doing` task with a different claimant: update `claimed_by:` and call it out, do not block.
- Reword the scope statements: README "What this is not" and CONTRIBUTING "Usually out of scope" must distinguish attribution and dependency metadata (in scope) from locking, leases, and scheduling runtimes (out of scope).
- Mention `claimed_by:` in the bootstrap block appended to `AGENTS.md`/`CLAUDE.md`.

## Done when
- `start` in `SKILL.md` writes and announces the claim.
- README and CONTRIBUTING scope statements no longer contradict the feature.

## Output
Updated `SKILL.md`, `README.md`, `CONTRIBUTING.md`.

## Dependencies
None.
