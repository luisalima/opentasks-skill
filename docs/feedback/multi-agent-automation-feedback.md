# opentasks — recommendations from a multi-agent automation pilot

Source feedback captured 2026-06-20. Intended as upstream feedback for the
`opentasks` skill/package. These are gaps in the `docs/tasks/` convention
surfaced while running a multi-agent task pipeline over a backlog with stacked
PRs and an unattended driver. Each item states the lesson, where it bit us, and
a concrete proposed change.

Tracking: filed as T11–T15 and folded into Q1. Item #3 (`depends_on`) was
already shipped as T2.

## 1. Multi-branch / PR-based workflows: where does task status live?

**Lesson.** opentasks assumes a single working copy where the task file and
`TASK_INDEX.md` are edited in place. With stacked feature PRs, every
implementation PR that flips a `TASK_INDEX` line touches the *same* lines, so the
PRs conflict with each other. We had to adopt a rule on the fly: implementation
PRs are code-only; status changes live on one branch.

**Proposed change.** Add a short "Working across branches" section to the
SKILL/README:
- Implementation PRs are **code-only** — never edit `docs/tasks/` inside a
  feature PR.
- Task status (`start`/`block`/`done`) is updated only on a single **tracking
  branch** (or `main`).
- During flight, treat **PR state as the live status** (open ≈ doing, merged ≈
  done) and reconcile the index in batch.

## 2. Add a first-class `autonomy:` frontmatter field

**Lesson.** To let a driver run tasks unattended we needed a per-task signal of
which tasks are safe to automate. We bolted on `autonomy: auto|human` and a
classification note in the index.

**Proposed change.** Make `autonomy` an optional frontmatter field:
- `autonomy: auto` — an agent/driver may execute this task unattended.
- `autonomy: human` (default when absent) — needs a person (open decision, too
  large, audit, or otherwise risky).
- Document the bar for `auto`: `status: todo`, dependencies satisfied, no linked
  open question, and a concrete machine-checkable `Done when` (see #4).
- `status` / `sync` can report the count of `auto`-eligible tasks.

## 3. Structured dependencies (`depends_on:`) → derivable ordering

**Lesson.** We wanted an explicit execution order/precedence. We added a manual
"Execution order" section to `TASK_INDEX.md`, but `sync` rebuilds the index and
would wipe it — we left a fragile "re-add after sync" caveat.

**Proposed change.** Add an optional `depends_on: [T2, T9]` frontmatter list
(distinct from freeform `links:`):
- `sync` derives and renders ordering from `depends_on` (topological), so
  precedence survives index rebuilds with no manual sections.
- The eligibility check in #2 can then verify "dependencies satisfied"
  structurally instead of by prose.

## 4. `Done when` must be independently verifiable — and adversarial for security

**Lesson.** An automation gate has to verify success *without trusting the
implementer*. Vague `Done when` ("implement X") blocks automation. For security
tasks the happy-path test is not enough; you need the negative/adversarial case
(the attack is *blocked*). This was the single most important readiness signal.

**Proposed change.** Strengthen the task template/guidance:
- `Done when` must be concrete and machine-checkable: a test command, an
  assertable contract, or an observable behavior — not a restatement of the work.
- Add explicit guidance: for security-relevant tasks, `Done when` must include
  the **adversarial/negative assertion** (rejected input, blocked request), not
  only the success path.
- Optionally add a `verify:` line (how to verify independently) to the template.

## 5. `sync` should preserve or derive curated sections

**Lesson.** We placed both the execution order and the autonomy classification in
`TASK_INDEX.md` as manual sections; `sync` would clobber them.

**Proposed change.** Prefer **derivation over preservation**: if `autonomy` (#2)
and `depends_on` (#3) are frontmatter fields, `sync` can regenerate the order and
the auto/human split deterministically, eliminating manual sections. If any
manual prose must remain, `sync` should preserve a clearly delimited region.

## 6. Link tasks that are spawned from a review/finding

**Lesson.** A security review *during* implementation surfaced a pre-existing
issue, and we opened a follow-up task for it.

**Proposed change (minor).** Add a convention to the "agents create tasks"
guidance: when a task is spawned from a review or finding, record the source
(report path, PR, or finding id) in `links:` so the provenance is traceable.
Already supported by the field; just make it a documented habit.

## 7. Reinforce: design-bearing tasks reference their open question

**Lesson.** Several tasks carried an unresolved design decision (e.g. an
authentication approach, a protocol/transport choice); we marked them `human`.
The `Q → ADR → T` flow exists, but the link from a blocked/human task back to its
open question was informal.

**Proposed change.** A task whose blocker is an open decision should reference the
question id (in `links:`, or a dedicated `blocked_by:`), and stay `autonomy:
human` until that question closes. This keeps the automation gate (#2) and the
decision flow consistent.

---

## Summary of proposed frontmatter additions

| Field | Type | Purpose |
|---|---|---|
| `autonomy` | `auto \| human` | gate unattended execution (#2) |
| `depends_on` | list of task ids | structured ordering + eligibility (#3) |
| `verify` | string (optional) | independent verification method (#4) |
| `blocked_by` | question id (optional) | tie a human task to its open decision (#7) |

The highest-leverage two are **`autonomy`** (#2) and **independently-verifiable
`Done when`** (#4): together they are exactly what an agent needs to decide,
unattended, whether a task is safe to run and whether it actually succeeded.
