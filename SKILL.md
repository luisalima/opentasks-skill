---
name: opentasks
description: "Maintain a lightweight docs/tasks/ repo convention for coding-agent tasks and open questions. Use when the user asks to bootstrap a tasks folder, create a task/question, start/block/close/reopen an item, list open tasks/questions, or sync the derived task index."
---

# /opentasks — docs/tasks/ convention manager

You are managing a lightweight repo convention in `docs/tasks/`, not an external task manager. The argument you received is:

**$ARGUMENTS**

Parse the intent and execute the appropriate operation below. If the argument is empty or ambiguous, run **status**.

Accept common aliases:
- `init` → `bootstrap`
- `create task`, `add task`, `task` → `new task`
- `open question`, `add question`, `question` → `new question`
- `close`, `finish`, `complete`, `mark done` → `done`
- `begin`, `work on` → `start`
- `pick up`, `reserve` → `claim`

Where an operation accepts `<item>`, resolve it before editing:
1. Exact filename under `docs/tasks/`, with optional `.md`.
2. Exact slug, meaning filename without `.md`.
3. Task shorthand `T<N>` / `t<N>` → the unique task with `id: T<N>` or `t<N>-*.md` filename.
4. Question shorthand `Q<N>` / `q<N>` → the unique `q<N>-*.md` file.
5. Exact case-insensitive title match from the file heading or index line.

If resolution finds no item, stop and tell the user what was searched. If it finds multiple items, stop and ask for the exact slug.

---

## Folder guard

Before executing any operation other than `bootstrap` / `init`, check whether `docs/tasks/` exists. If it does not, stop and tell the user:

> No `docs/tasks/` folder found. Run `/opentasks bootstrap` to set up the system first.

---

## Task sizing and agent behavior

A task should be small enough for one focused agent session or one coherent PR. Good tasks have one objective, concrete `Done when` criteria, independent verification, and no unresolved design choice hidden inside the scope.

`Done when` must be **independently checkable** — it states how someone *other than the implementer* confirms success, so it must not merely restate the work ("implement X"). The verification mechanism is repo-dependent and there is no single required form: a test command, a lint or validation run, an assertable contract, or an observable artifact or behavior. For security-relevant tasks, `Done when` must include the **adversarial/negative** assertion (malicious input rejected, unauthorized request blocked), not only the happy path — the negative case is the load-bearing signal. When a concrete command or method exists, record it in the optional `verify:` frontmatter field; the field is a convenience pointer, not a requirement.

Split a task when it has multiple outputs, multiple owners, unresolved decisions, or a title that naturally contains "and then." Do not create tasks for every tiny edit. Create them when work needs to survive chat context, coordinate across humans or agents, or show up in git history.

Agents should create tasks when breaking down a user-approved plan, discovering follow-up work that should not be done immediately, finding a blocker or dependency, extracting implementation work from an ADR, or leaving continuation work for another human or agent. Agents should not create tasks merely to describe work they are already completing in the same turn.

## ADRs and decision flow

Use questions for unresolved decisions, ADRs for durable decisions, and tasks for execution:

```text
Q<N> -> ADR -> T<N>
```

- Open `Q<N>` when a decision is unresolved.
- Create or update an ADR when the answer has architectural or long-lived consequences.
- Close the question with the decision and link to the ADR.
- Create tasks for the implementation work that follows from the ADR.
- Link ADR-derived tasks back to the ADR using `links:`.
- If a task uncovers a durable decision, open a question and block or split the task until the ADR resolves it.
- If implementation shows the ADR is wrong or incomplete, open a new question instead of silently changing task scope.
- A task whose blocker is an open decision references that question in `blocked_by: Q<N>` and stays `autonomy: human` until the question closes — this keeps the decision flow and the automation gate consistent. `blocked_by` is the normative decision link; freeform `links:` stays for general provenance.

---

## Operations

### `bootstrap` / `init`
Set up a fresh `docs/tasks/` folder in a new project.
1. Ask the user for the project's deliverable names if not obvious from context (e.g. D1, D2, ops).
2. Confirm what language the project documentation is in (default to English if obvious).
3. Create `docs/tasks/README.md` — a human-readable conventions doc. See **§README** below for required content.
4. Create `docs/tasks/TASK_INDEX.md` — empty index with group headers for each deliverable + Open questions section. See **§INDEX** below for format.
5. Do NOT create any tasks or questions yet.
6. Append the following block to `AGENTS.md` (create it if absent) only if it does not already contain a `## Task and question tracking` section. If `CLAUDE.md` exists and does not already contain that section, append the same block there too.

```markdown
## Task and question tracking

This project uses `docs/tasks/` as a lightweight repo convention for work items and open decisions. Use the `/opentasks` skill to manage it.

- When planning or breaking down work, record concrete steps as tasks (`/opentasks new task <title>`) and open decisions as questions (`/opentasks new question <title>`).
- Keep tasks sized for one focused agent session or one coherent PR. Split work with multiple outputs, owners, or unresolved decisions.
- Use questions for unresolved decisions, ADRs for durable decisions, and tasks for execution; link ADR-derived tasks back to the ADR in `links:`.
- Keep status current: mark items `doing` when you start, `blocked` when waiting, `done` when complete.
- `/opentasks start <item>` claims a task and begins the work in the same turn; `/opentasks claim <item>` records the claim (`claimed_by: <who> @ <where>`) without starting. Claims are attribution, not locks.
- Never create task or question files manually — always go through `/opentasks` to keep the index in sync.
```

---

### `new task <title>`
Create a new task file.
1. Find the next task number N: scan existing `t<N>-*.md` files and task frontmatter `id: T<N>`, take max N + 1 (start at 1 if none exist). Task IDs are monotonic and never reused.
2. Generate a kebab-case slug from the title (ASCII, lowercase, hyphens; strip accents, replace spaces with hyphens).
3. Check task size before creating it. If the title or context implies multiple outputs, multiple owners, unresolved decisions, or "and then" sequencing, split it into smaller tasks or create a question for the unresolved decision first.
4. If `docs/tasks/t<N>-<slug>.md` already exists, abort and tell the user the file exists — suggest they pick a different title or run `/opentasks status` to see the existing item.
5. Determine the deliverable bucket from context or ask the user. If the title or context signals urgency or explicit ordering, also set `priority: p1` (do first) or `priority: p3` (can wait); otherwise omit the field — absent means `p2`.
6. If the task comes from an ADR, include the ADR path in `links:`. If the work waits on other tracked tasks, list their IDs in `depends_on:`.
7. Write `docs/tasks/t<N>-<slug>.md` with this exact structure:

```markdown
---
status: todo
type: task
id: T<N>
deliverable: <D1|D2|ops|…>
created: <YYYY-MM-DD>
links: []
---

# T<N>. <Title>

## Objective
<One or two sentences — what this is and why it matters.>

## What we need to extract / do
<Concrete bullets describing the actual work.>

## Done when
- <An independently-checkable criterion — a command, a contract, or an observable behavior, in whatever form the repo supports; not a restatement of the work. For security-relevant work, include the adversarial/negative case (input rejected, request blocked), not only the success path.>

## Output
<What gets produced and where it feeds into. Write "none" if the task produces no tracked artifact.>

## Dependencies
<What must exist first.>
```

8. For ADR-derived tasks, replace `links: []` with a YAML list containing the ADR path.
9. Add a line to `TASK_INDEX.md` under the correct `## <Deliverable>` group:
   `- [ ] [T<N>. <Title>](t<N>-<slug>.md) — \`todo\``
   For `p1` and `p3` tasks, append the priority tag after the status: `` — `todo` `p1` ``.

---

### `new question <title>`
Create a new question file.
1. Find the next question number N: scan existing `q<N>-*.md` files, take max N + 1 (start at 1 if none exist).
2. Generate a kebab-case slug from the title.
3. Ask the user for `owner` if not clear from context (who needs to answer — a person or role).
4. Write `docs/tasks/q<N>-<slug>.md`:

```markdown
---
status: todo
type: question
owner: <name-or-role>
created: <YYYY-MM-DD>
---

# Q<N>. <The question, phrased as a question>

**Why it matters:** <impact on design / scope>
- Branch A → consequence A.
- Branch B → consequence B.

**Still open:** <what remains unclear>
```

5. Add a line to `TASK_INDEX.md` under `## Open questions`, sub-grouped by owner:
   `- [ ] [Q<N>. <Title>](q<N>-<slug>.md) — \`todo\``

---

### `claim <item>`
Record that someone is picking a task up, without beginning the work — e.g. reserving it for a later session, another checkout, or another person.
1. Resolve `<item>` and open the matching file.
2. If the file has `type: question` in frontmatter, abort and tell the user: questions don't use `doing` status — use `/opentasks block` to flag as waiting, or `/opentasks done` to record the answer.
3. If `status: done`, abort and tell the user to run `/opentasks reopen <item>` first.
4. Change `status: todo` or `status: blocked` → `status: doing` in frontmatter. If it is already `doing`, leave it as-is.
5. Add `started: <YYYY-MM-DD>` to frontmatter directly after `status` if absent. Do not duplicate or overwrite an existing `started:`.
6. Set `claimed_by: <who> @ <where>` in frontmatter — `<who>` is the agent or person claiming the task (e.g. `claude-code`, `luisa`), `<where>` is the host, checkout, or session it is running in. If the task is already `doing` with a different `claimed_by`, overwrite it and call out the takeover in your reply — a claim is attribution, not a lock; never refuse on the basis of an existing claim.
7. Announce the claim in your reply, e.g. "Claimed T4 as claude-code on lu-macbook."
8. If a `## Blocker` section is present and the blocker is no longer relevant, remove it or add a short resolved note.
9. Update the index line: replace the current status tag with `` `doing` `` and remove any `(waiting on ...)` suffix.

---

### `start <item>`
Claim the task, then begin working on it in the same turn. `start` = `claim` + execution; use `claim` when the work should not begin yet.
1. Perform every step of `claim <item>` above.
2. Then begin executing the task immediately — do not stop after announcing the claim. Read the task body and carry out `## What we need to extract / do`, working toward the `## Done when` criteria.
3. If execution hits a blocker, a missing dependency, or an unresolved decision, record it (`block <item>`, or a new question) and report what is needed rather than silently stopping.
4. If the `## Done when` criteria are met within the session, close the task via the `done <item>` flow.

---

### `block <item> [reason]`
1. Resolve `<item>` and open the matching file.
2. If `status: done`, abort and tell the user to run `/opentasks reopen <item>` first.
3. Change status → `status: blocked` in frontmatter. If it is already `blocked`, leave it as-is.
4. Add or update a single `## Blocker` section in the body explaining what's being waited on. If no reason was provided and it is not obvious from context, ask for the blocker.
5. Update the index line: replace the current status tag with `` `blocked` `` and append `(waiting on <reason>)` if a reason was given.

---

### `done <item>`
1. Resolve `<item>` and open the matching file.
2. If the item is already `done`, leave existing answer/output/closed fields unchanged unless the user explicitly asks to correct them, then skip to the `TASK_INDEX.md` update.
3. Before editing, validate the closure:
   - **If it's a question:** if the answer is not clear from context, ask for it before closing.
   - **If it's a task:** check the `## Done when` section. If the criteria are clearly unmet or ambiguous, stop and report what remains unless the user explicitly asked to close it anyway.
4. Set `status: done`, add `closed: <YYYY-MM-DD>` to frontmatter if absent, and do not duplicate an existing `closed:`.
5. **If it's a question:** add the answer inline in the body with date and source. Do not replace the question text.
6. **If it's a task that produced a tracked artifact:** add or update `output: <path>` in frontmatter. Omit `output:` entirely if the task produced no tracked artifact.
7. In `TASK_INDEX.md`:
   - Flip `[ ]` → `[x]` on the item's line.
   - Replace status tag with `` `done` ``.
   - For tasks with an output: append `→ <output-path>`.
   - For questions: move the line to the `**Answered (history):**` sub-group under `## Open questions`.

---

### `reopen <item>`
Reopen a previously closed item.
1. Resolve `<item>` and open the matching file.
2. Set `status: todo` in frontmatter.
3. Remove the `closed:` field from frontmatter. For tasks, also remove `output:` because the prior output is no longer the current completion artifact. Leave `started:` in place if present — it records prior work.
4. In `TASK_INDEX.md`:
   - Flip `[x]` → `[ ]` on the item's line.
   - Replace `` `done` `` with `` `todo` ``.
   - Remove any `→ output-path` suffix from the line.
   - For questions: move the line back from `**Answered (history):**` to the active sub-group under `## Open questions`.

---

### `list [filter]`
Read `docs/tasks/` and print a filtered view. Accepts one optional argument:

| Filter | What it shows |
|---|---|
| _(none)_ | All non-done items, grouped by deliverable / owner |
| `todo` / `doing` / `blocked` / `done` | Items with that exact status |
| `<deliverable>` (e.g. `D1`, `ops`) | All items in that deliverable bucket |
| `questions` | All question files regardless of status |
| `p1` / `p2` / `p3` | Tasks with that priority (absent counts as `p2`) |

Output format: same grouped list style as the index — one item per line with checkbox and status tag. Print a count summary at the bottom.

---

### `next [deliverable]`
Recommend the next task to pick up — answers "what should I work on?" deterministically. Read-only: it changes no files and claims nothing.
1. Read frontmatter from every task file. If a deliverable argument is given (e.g. `next D1`), consider only tasks in that bucket.
2. Collect the **ready** tasks: `status: todo` with every ID in `depends_on` `done` (see **Dependencies and readiness**).
3. Pick the highest-priority ready task (`p1` before `p2` before `p3`; absent counts as `p2`). Break ties by lowest task number.
4. Report the pick: ID, title, file, and why it was chosen — its priority, which dependencies are satisfied, and whether it is auto-eligible (`autonomy: auto` with no open `blocked_by`) or needs a human. List the runners-up briefly, one line each, in the same priority-then-number order. Suggest `/opentasks start T<N>` to claim it and begin, or `/opentasks claim T<N>` to just reserve it.
5. If no task is ready, say so, and show the highest-priority non-ready open task (same ordering) with what blocks it: its unmet `depends_on` IDs with their statuses, or its `## Blocker` section if it is `blocked`. If there are no open tasks at all, say so.

---

### `graph [write]`
Visualize the task dependency graph from `depends_on` as a Mermaid flowchart. GitHub renders Mermaid natively. Read-only unless `write` is given.
1. Read frontmatter from every task file: `id`, `status`, `depends_on`, and the title from the `# T<N>.` heading.
2. If no task has a non-empty `depends_on`, print "No `depends_on` relationships found — nothing to graph." and stop. Never emit an empty diagram.
3. Build a fenced ` ```mermaid ` block:
   - First line: `graph TD`.
   - Include only tasks that appear on either side of a dependency edge; leave unconnected tasks out to keep the diagram readable.
   - One node per included task: `T5["T5. <Title>"]:::<status>` — always double-quote the label and strip backticks and double quotes from the title.
   - One edge per dependency, pointing from prerequisite to dependent: `depends_on: [T2]` on T5 emits `T2 --> T5`.
   - A `depends_on` ID with no matching task file still gets a node, labelled `T9["T9 (missing)"]:::missing` — `sync` and `status` report it as a mismatch.
4. Style nodes by status by appending these classDefs at the end of the block — done greyed out, doing highlighted, blocked marked with a dashed red border:

```text
classDef todo fill:#ffffff,stroke:#495057
classDef doing fill:#fff3bf,stroke:#f59f00,stroke-width:2px
classDef blocked fill:#ffe3e3,stroke:#e03131,stroke-dasharray: 5 5
classDef done fill:#e9ecef,stroke:#adb5bd,color:#868e96
classDef missing fill:#ffffff,stroke:#e03131,stroke-dasharray: 2 2,color:#e03131
```

5. Output destination:
   - **Default:** print the Mermaid block to the user. No file is touched.
   - **`graph write`:** also write the block into `TASK_INDEX.md` under a `## Dependency graph` section at the end of the file, replacing that section if it already exists. Once the section exists, `sync` regenerates it on every run; delete the section to stop maintaining it.

---

### `sync`
Rebuild `TASK_INDEX.md` from scratch by reading all files in `docs/tasks/`.
1. Read frontmatter from every `*.md` file except `README.md` and `TASK_INDEX.md`.
2. Group `type: task` files by `deliverable`; group `type: question` files by `owner`. Prefer the task `id` field for labels and sorting; fall back to the filename only for legacy task files.
3. For backward compatibility, treat files with no `type` and a `deliverable` as tasks, but report them as frontmatter mismatches. Also report task files missing `id: T<N>` as legacy task files that should be migrated, and report `priority` values outside `p1`/`p2`/`p3` as mismatches. Validate `depends_on`: unknown task IDs, self-references, and cycles are mismatches. Validate `blocked_by`: an unknown question ID is a mismatch. Validate `autonomy`: a value other than `auto`/`human` is a mismatch, as is `autonomy: auto` on a task with an unanswered `blocked_by`.
4. Emit the full index in the format defined in **§INDEX**.
5. Frontmatter is the source of truth — discard the old index content entirely, with one exception: if the old index had a `## Dependency graph` section (created by `graph write`), regenerate it with the rules from **`graph`** and keep it at the end of the file.

---

### `migrate`
Upgrade a legacy `docs/tasks/` folder to the current convention in place. Idempotent — running it on an up-to-date folder changes nothing.
1. Read every `*.md` file except `README.md` and `TASK_INDEX.md`.
2. For each file, apply only the upgrades it is missing:
   - No `type` but a `deliverable` → add `type: task`. No `type` but an `owner` → add `type: question`. Neither → report the file as ambiguous; do not guess.
   - Task missing `id:` → take `T<N>` from a `t<N>-*` filename if present; otherwise allocate the next free number (monotonic, never reused).
   - Task file not named `t<N>-<slug>.md` → rename it to match its `id` (use `git mv` in a git repo).
   - Task missing `links:` → add `links: []`.
3. Run `sync` to rebuild the index.
4. Report every change made, file by file. If nothing needed upgrading, say so.

When a future convention change affects existing files, its upgrade steps must be added to this operation in the same PR (see CONTRIBUTING).

---

### `status` (default when no argument given)
Read `TASK_INDEX.md` (or scan `docs/tasks/` if the index is absent) and report:
- Count of items by status: todo / doing / blocked / done.
- Count of auto-eligible tasks (`autonomy: auto`, ready, and no open `blocked_by`).
- All `doing` and `blocked` items with their blockers.
- Any frontmatter/index mismatches spotted, including missing `type`, invalid statuses, questions marked `doing`, tasks missing `id`, tasks missing `deliverable`, questions missing `owner`, duplicate task IDs, duplicate question numbers, malformed `links`, invalid `priority` values, invalid `depends_on` references (unknown IDs, self-references, cycles), `doing` tasks whose dependencies are not `done`, invalid `blocked_by` references (unknown question IDs), invalid `autonomy` values (anything other than `auto`/`human`), tasks marked `autonomy: auto` with an unanswered `blocked_by`, and stale index lines.

---

## File naming rules

- Task slugs: `t<N>-<short-slug>`. Examples: `t1-llm-shortlist`, `t8-extract-analysis-scripts`.
- Question slugs: `q<N>-<short-slug>`. Examples: `q1-classification-rubric`, `q7-template-wording`.
- All filenames are lowercase ASCII — strip accents, replace spaces with hyphens.
- Task and question numbers are monotonic across the project lifetime. Never reuse a number.

---

## Status semantics

| Status    | Tasks                        | Questions                      |
|-----------|------------------------------|--------------------------------|
| `todo`    | Not started                  | Ready to ask / discuss         |
| `doing`   | In progress                  | Not valid — use `block` or `done` directly |
| `blocked` | Waiting on dependency        | Waiting for an answer          |
| `done`    | Completed                    | Answered                       |

---

## Dependencies and readiness

Tasks may declare machine-readable dependencies in `depends_on:` — a YAML list of task IDs, e.g. `depends_on: [T3, T7]`. The `## Dependencies` body section stays free-text context; `depends_on` is the normative list.

A task is **ready** when `status: todo` and every task in `depends_on` is `done`. The `next` operation picks among ready tasks.

`depends_on` tracks prerequisite *tasks*; the separate optional `blocked_by: Q<N>` points at an open *question* (a decision) that gates the task. A task with an unanswered `blocked_by` stays `autonomy: human` and is not eligible for unattended execution.

`sync` and `status` validate `depends_on`: unknown task IDs, self-references, and dependency cycles are frontmatter mismatches. `status` also flags `doing` tasks whose dependencies are not all `done`.

---

## Autonomy and unattended execution

`autonomy:` marks whether an agent or driver may execute a task without a human in the loop:

- `autonomy: auto` — safe to pick up and run unattended.
- `autonomy: human` — needs a person: an open decision, an audit, or something too large or risky. **`human` is the default** when the field is absent.

A task is **auto-eligible** only when all of these hold:

- `autonomy: auto`;
- `status: todo` and it is **ready** (every `depends_on` is `done`);
- no open decision gates it (`blocked_by` absent, or its question is `done`);
- its `Done when` is concrete and independently checkable (see [task sizing and agent behavior](#task-sizing-and-agent-behavior)) — including the adversarial/negative assertion for security-relevant tasks.

The first three conditions are structural and checked by `sync`/`status`; the last is a judgment the task's author makes before setting `autonomy: auto`. `status` reports the count of auto-eligible tasks, and `next` notes whether its recommended task is auto-eligible. Setting `autonomy: auto` on a task with an unanswered `blocked_by` is a mismatch — resolve the decision or keep it `human`.

---

## Frontmatter reference

**Tasks:**
```yaml
---
status: todo            # todo | doing | blocked | done
type: task
id: T<N>                # stable task identifier, monotonic and never reused
deliverable: D2         # project-specific bucket
created: YYYY-MM-DD
links: []               # optional related URLs or repo paths
priority: p2            # optional: p1 | p2 | p3; treated as p2 when absent
autonomy: human         # optional: auto | human; absent means human (needs a person)
depends_on: []          # optional list of task IDs this task waits on, e.g. [T3, T7]
verify: <command>       # optional: how to confirm completion independently; form depends on the repo
blocked_by: Q<N>        # optional: an open question that gates this task; keep autonomy: human until it closes
started: YYYY-MM-DD     # added by `claim`/`start`; kept on reopen as historical record
claimed_by: who @ where # added by `claim`/`start`; attribution, not a lock
closed: YYYY-MM-DD      # only when status = done; removed by `reopen`
output: path/to/file.md # only if the task produced a tracked artifact
---
```

**Questions:**
```yaml
---
status: todo            # todo | blocked | done  (never doing)
type: question
owner: <name-or-role>
created: YYYY-MM-DD
closed: YYYY-MM-DD      # only when status = done; removed by `reopen`
---
```

---

## §README — what `docs/tasks/README.md` must cover

Write the README in the same language as the project's documentation. It must include:

1. One-paragraph intro: this folder is a lightweight repo convention for both execution tasks and open questions using flat markdown + YAML frontmatter; item type is distinguished by the `type` frontmatter field.
2. How it works: one file per item; frontmatter is the source of truth; `TASK_INDEX.md` is a derived view; tasks have stable `T<N>` identifiers.
3. The four status values and what they mean for tasks vs questions (see table above).
4. Type conventions: task vs question; what `owner` is for; what `links` is for; the optional `priority` field (`p1`/`p2`/`p3`, default `p2`); the optional `depends_on` list and the readiness rule (a task is ready when `todo` and all dependencies are `done`); the optional `verify:` field (how to confirm completion independently, form repo-dependent); the optional `blocked_by:` field (a question id that gates the task); the optional `autonomy:` field (`auto`/`human`, default `human`) and the bar a task clears to be `auto`-eligible.
5. Task sizing guidance: one focused agent session or one coherent PR; split work with multiple outputs, owners, unresolved decisions, or "and then" sequencing. `Done when` must be independently checkable rather than a restatement of the work, and must include the adversarial/negative assertion for security-relevant tasks.
6. Agent creation guidance: create tasks for user-approved plans, deferred follow-up work, blockers, ADR implementation work, and handoffs; do not create tasks merely to describe same-turn work.
7. ADR and decision flow: `Q<N> -> ADR -> T<N>`, plus the reverse path when a task uncovers a durable decision; ADR-derived tasks link back to the ADR in `links:`.
8. The full task body template as a fenced markdown block.
9. The full question body template as a fenced markdown block.
10. Workflow: create → claim/start → block → close → reopen, with the exact frontmatter changes at each step, including that `start` claims and then begins the work while `claim` only records the claim.
11. A note that closed files are kept as history — never delete.
12. A short note that this is not a task manager, Kanban board, daemon, database, sync service, or UI.

---

## §INDEX — `TASK_INDEX.md` format

```markdown
> Frontmatter is the source of truth. This index is a derived view — if they disagree, read the individual .md files.

## D1 — <Deliverable name>

- [ ] [T1. <Title>](t1-slug.md) — `todo`
- [ ] [T2. <Title>](t2-slug.md) — `blocked` (waiting on client data)
- [x] [T3. <Title>](t3-slug.md) — `done` → path/to/output.md

## D2 — <Deliverable name>

…

## Open questions

**For <person A>:**
- [ ] [Q1. <Question>](q1-slug.md) — `todo`

**For <person B>:**
- [ ] [Q2. <Question>](q2-slug.md) — `blocked`

**Answered (history):**
- [x] [Q3. <Question>](q3-slug.md) — `done`
```

Rules:
- `[x]` only when `status: done`. All other statuses use `[ ]` (including `blocked`).
- Task lines include the stable task ID in the visible label, e.g. `T4. Implement cache`.
- `blocked` items may have a parenthetical explanation: `(waiting on client data)`.
- Non-default priorities appear as a tag after the status: `` `todo` `p1` ``. Default `p2` is never shown.
- Append `→ path/to/output` for `done` tasks that produced a tracked artifact.
- Closed files are never deleted — they remain in the folder and in the index as history.
- An optional `## Dependency graph` section (created by `graph write`) may close the file; `sync` regenerates it from `depends_on` frontmatter.
