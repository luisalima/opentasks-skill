# ADR-0001: Agent self-dispatch — priority, dependencies, and claim attribution

- **Status:** accepted
- **Date:** 2026-06-11

## Context

OpenTasks v0.1.0 deliberately excludes multi-agent scheduling: the README declares it "not a multi-agent scheduler or claiming protocol" and CONTRIBUTING lists "multi-agent scheduling, locking, or claiming protocols" as out of scope. In practice, three recurring needs push against that line:

1. **"What should I work on next?"** — there is no priority field and no ordering signal; the index is grouped by deliverable and ordered by task number only.
2. **"Who picked this up, and where?"** — `start` flips status and stamps `started:`, but records nothing about which human or agent claimed the task or in which environment it is running.
3. **"What depends on what?"** — the `## Dependencies` body section and `links:` are free text, not machine-readable, so no graph, no ready-task detection, and no automatic blocked-flagging is possible.

These are one feature family: together they are what lets an agent dispatch itself from the folder instead of being hand-fed tasks.

## Decision

Add three optional frontmatter fields and two operations, keeping everything flat markdown with no runtime:

- **`priority:`** — `p1` | `p2` | `p3` (default `p2` when absent). Shown as a tag on index lines.
- **`depends_on:`** — YAML list of task IDs, e.g. `depends_on: [T3, T7]`. `sync`/`status` validate unknown IDs and cycles, and flag tasks whose dependencies are not `done`. A task is **ready** when `status: todo` and all `depends_on` are `done`.
- **`claimed_by:`** — written by `start`: who/what claimed the task (agent name or person), and where it is running (host, repo checkout, or session). The agent must announce the claim in its reply when starting a task.
- **`next` operation** — prints the ready task with the highest priority (ties broken by lowest task number); answers "what should I pick up?" deterministically.
- **`graph` operation** — emits a Mermaid diagram of the `depends_on` graph (GitHub renders Mermaid natively).

**Attribution, not locking.** `claimed_by:` is a record, not a lease: there is no expiry, no conflict resolution, and no enforcement. Git merge remains the arbiter, exactly as for every other field. If two agents claim the same task, the merge conflict *is* the signal.

## Consequences

- The scope statements in README ("What this is not") and CONTRIBUTING ("Usually out of scope") must be reworded to distinguish *attribution and dependency metadata* (in scope) from *locking, leases, and scheduling runtimes* (still out of scope).
- The index line format, frontmatter reference, `sync`, and `status` in both `SKILL.md` and `README.md` gain the new fields and validations.
- All new fields are optional, so existing `docs/tasks/` folders remain valid without migration (see ADR-0003 for the migration mechanism regardless).
- Implementation tasks: T1 (priority), T2 (depends_on), T3 (next), T4 (claimed_by + scope rewording), T5 (graph).
