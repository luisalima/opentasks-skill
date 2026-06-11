# ADR-0003: Convention evolution and migration

- **Status:** accepted
- **Date:** 2026-06-11

## Context

The convention evolves: v0.1.0 introduced `type:`, stable `id: T<N>`, and `links:`, and ADR-0001 adds more fields. Two update paths are currently unhandled:

1. **Updating the skill installation.** Installation is a git clone into `~/.claude/skills/opentasks` (or a project-level path), but the README never says how to update it. Stale installs silently apply an old convention — this happened in this very repo: the globally installed skill predates v0.1.0 and would create tasks without IDs.
2. **Updating existing `docs/tasks/` folders.** `sync` *detects* legacy files (missing `type`, missing `id`) and reports them, but nothing fixes them. CONTRIBUTING asks PRs to "describe any migration impact" with no mechanism to act on it.

## Decision

- Add a **`migrate` operation** to the skill: it upgrades legacy frontmatter in place — adding missing `type`, assigning `id: T<N>` from filenames or by allocating the next free number, normalizing fields renamed by later versions — then runs `sync` and reports every change made. It is idempotent: running it on an up-to-date folder changes nothing.
- Each future convention change that affects existing files must ship with its migration steps added to the `migrate` operation in the same PR.
- Document in the README's Installation section that updating the skill is `git pull` in the clone directory, and that `/opentasks migrate` upgrades existing folders after a skill update.

## Consequences

- `sync` stays read-only/detect; `migrate` is the only operation that rewrites files it did not just create.
- The CONTRIBUTING "migration impact" checklist item becomes actionable: impact must be encoded in `migrate`, not just described.
- Implementation tasks: T6 (migrate operation), T10 (update instructions in README).
