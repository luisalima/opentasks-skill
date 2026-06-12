---
status: todo
type: question
owner: luisa
created: 2026-06-12
---

# Q1. Should task state live on a branch that is always pushed, and how?

**Why it matters:** `claimed_by` and status changes only coordinate across machines and agents if they are visible remotely. Today task files live on whatever branch you happen to be on, so a claim made in one checkout is invisible elsewhere until that branch is pushed and merged.
- Branch A → dedicated always-pushed `tasks` branch (possibly orphan): every `/opentasks` mutation commits and pushes there. State is visible everywhere immediately, but task files diverge from feature branches, and every operation grows commit/push/conflict handling.
- Branch B → keep tasks on `main`, auto-commit and auto-push task-file changes after each mutation. Simple mental model, but pushes to `main` on every status flip and requires task commits to stay separate from code commits.
- Branch C → status quo: task state syncs when branches merge; claims are attribution, not locks, so eventual consistency may be acceptable.

**Still open:** Is the cross-checkout coordination need real enough to justify auto-push? How are concurrent edits to the same task file reconciled? This is a durable workflow decision — likely needs an ADR before any implementation task.
