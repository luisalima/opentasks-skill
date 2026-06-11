---
status: done
type: task
id: T10
deliverable: D3
created: 2026-06-11
links:
  - docs/adr/0003-convention-migration.md
started: 2026-06-11
claimed_by: claude-code @ the-tasks-folder
closed: 2026-06-11
output: README.md
---

# T10. Document how to update an installed skill

## Objective
Installation is a git clone, but the README never says how to update it. Stale installs silently apply an old convention — the globally installed skill on this machine predates v0.1.0.

## What we need to extract / do
- Add an "Updating" note to the README Installation section: `git pull` in the clone directory (`~/.claude/skills/opentasks`, `~/.codex/skills/opentasks`, or the project-level path), then restart Codex if applicable.
- Mention running `/opentasks migrate` in existing projects after updating, once T6 lands.
- Suggest checking the installed version against the repo (e.g. `git -C ~/.claude/skills/opentasks log -1`).

## Done when
- README Installation covers updating for all three install targets.

## Output
Updated `README.md`.

## Dependencies
None (the `migrate` cross-reference can land as "coming" or follow T6).
