# docs/tasks/ — task and question tracking

This folder is a lightweight repo convention for tracking both execution tasks and open questions as flat markdown files with YAML frontmatter. Item type is distinguished by the `type` frontmatter field (`task` or `question`). It is the OpenTasks convention, applied to the OpenTasks repo itself.

## How it works

- One file per item, flat in this folder — no subfolders, no hidden state.
- Frontmatter is the source of truth. `TASK_INDEX.md` is a derived view; if they disagree, the individual files win.
- Tasks have stable `T<N>` identifiers, monotonic across the project lifetime and never reused.
- Manage items through the `/opentasks` skill rather than editing by hand, so the index stays in sync.

## Status values

| Status    | Tasks                        | Questions                                  |
|-----------|------------------------------|--------------------------------------------|
| `todo`    | Not started                  | Ready to ask / discuss                     |
| `doing`   | In progress                  | Not valid — use `block` or `done` directly |
| `blocked` | Waiting on dependency        | Waiting for an answer                      |
| `done`    | Completed                    | Answered                                   |

## Item types

- **Tasks** (`type: task`) track work. They carry an `id: T<N>`, a `deliverable` bucket, and optional `links:` to related URLs, repo paths, or ADRs.
- **Questions** (`type: question`) track unresolved decisions. They carry an `owner` — the person or role who needs to answer.
- `links:` records related issues, PRs, docs, branches, or ADR paths. ADR-derived tasks link back to their ADR here.

## Task sizing

A task should be small enough for one focused agent session or one coherent PR. Good tasks have one objective, concrete `Done when` criteria, independent verification, and no unresolved design choice hidden inside the scope.

Split a task when it has multiple outputs, multiple owners, unresolved decisions, or a title that naturally contains "and then." Do not create tasks for every tiny edit — create them when work needs to survive chat context, coordinate across humans or agents, or show up in git history.

## When agents create tasks

Agents should create tasks when breaking down a user-approved plan, discovering follow-up work that should not be done immediately, finding a blocker or dependency, extracting implementation work from an ADR, or leaving continuation work for another human or agent. Agents should not create tasks merely to describe work they are already completing in the same turn.

## ADRs and decision flow

Use questions for unresolved decisions, ADRs for durable decisions, and tasks for execution:

```text
Q<N> -> ADR -> T<N>
```

- Open `Q<N>` when a decision is unresolved.
- Create or update an ADR (in `docs/adr/`) when the answer has architectural or long-lived consequences.
- Close the question with the decision and link to the ADR.
- Create tasks for the implementation work that follows from the ADR, linking back to the ADR in `links:`.
- If a task uncovers a durable decision, open a question and block or split the task until the ADR resolves it.
- If implementation shows the ADR is wrong or incomplete, open a new question instead of silently changing task scope.

## Task body template

```markdown
---
status: todo
type: task
id: T<N>
deliverable: <D1|D2|D3>
created: <YYYY-MM-DD>
links: []
---

# T<N>. <Title>

## Objective
<One or two sentences — what this is and why it matters.>

## What we need to extract / do
<Concrete bullets describing the actual work.>

## Done when
- <Concrete, observable completion criterion.>

## Output
<What gets produced and where it feeds into. Write "none" if the task produces no tracked artifact.>

## Dependencies
<What must exist first.>
```

## Question body template

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

## Workflow

- **create** — file written with `status: todo` and `created:` date; line added to the index.
- **claim** (tasks only) — `status:` → `doing`; `started: <date>` added if absent; `claimed_by: <who> @ <where>` records who picked it up, without beginning the work.
- **start** (tasks only) — does everything `claim` does, then begins executing the task in the same turn.
- **block** — `status:` → `blocked`; a `## Blocker` section explains what's being waited on; index line gets `(waiting on …)`.
- **done** — `status:` → `done`; `closed: <date>` added; tasks that produced an artifact get `output: <path>`; questions get the answer recorded inline with date and source.
- **reopen** — `status:` → `todo`; `closed:` (and `output:` for tasks) removed; `started:` kept as historical record.

Closed files are kept as history — never delete them.

## Limitations

Task state is **eventually consistent**. It lives in git, so what you see is your checkout's view: a status change or claim made on another branch or machine is invisible until that branch is pushed and merged. Consequences:

- `claimed_by` is attribution, not a lock — two agents on different checkouts can claim the same task without either noticing. The claim tells you who to talk to, not who has exclusive rights.
- The index and frontmatter can both be "correct" yet stale relative to work elsewhere; convergence happens at merge time, like any other file in the repo.
- Conflicting edits to the same task file are resolved as ordinary git merge conflicts.

If tighter coordination is ever needed, that's a workflow decision, not a convention tweak (tracked as [Q1](q1-tasks-on-always-pushed-branch.md)).

This folder is not a task manager, Kanban board, daemon, database, sync service, or UI. It is markdown in git.
