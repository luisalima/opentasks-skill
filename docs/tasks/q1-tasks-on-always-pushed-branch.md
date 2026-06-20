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
- Branch D → status lives on one tracking branch (or `main`) only: implementation PRs are **code-only** and never touch `docs/tasks/`; during flight, PR state *is* the live status (open ≈ doing, merged ≈ done) and the index is reconciled in batch. Avoids the stacked-PR conflict below without a daemon.

**Newly surfaced (multi-agent automation pilot, see [feedback #1](../feedback/multi-agent-automation-feedback.md)):** with stacked feature PRs, every implementation PR that flips a `TASK_INDEX.md` line edits the *same* lines, so the PRs conflict with each other. The pilot adopted Branch D on the fly (code-only PRs; status on one branch). This makes the choice more urgent than pure cross-checkout visibility.

**Still open:** Is the cross-checkout coordination need real enough to justify auto-push (A/B), or is the lighter "code-only PRs + status on one branch + PR-state-as-status" rule (D) sufficient? How are concurrent edits to the same task file reconciled? This is a durable workflow decision — record the answer as an ADR, and if it lands on Branch D, the implementation task is a short "Working across branches" section in `SKILL.md`/README.
