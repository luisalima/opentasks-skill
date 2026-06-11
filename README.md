# opentasks-skill

A tiny repo convention and Agent Skills-compatible workflow for tracking coding-agent tasks and open questions in `docs/tasks/`.

OpenTasks is not a task manager, Kanban board, daemon, database, sync service, or UI. It is a small Markdown/YAML convention that lets humans and coding agents keep durable tasks and unresolved decisions visible in git.

The convention tracks both **work items** and **open questions** using one markdown file per item. The `type` field distinguishes tasks from questions, tasks use stable `T<N>` identifiers, frontmatter is the source of truth, and `TASK_INDEX.md` is a derived view.

## Best for

- Small and medium repos
- Agent-assisted implementation work
- Short project plans that should survive chat context
- Open decisions that should not disappear in conversation
- Teams that want task state visible in normal git diffs

## What this is not

- Not a replacement for GitHub Issues, Linear, Jira, or Taskwarrior
- Not a personal productivity system
- Not a calendar/task sync format
- Not a Kanban app
- Not a multi-agent scheduler — claims (`claimed_by:`) are attribution, not locks: no leases, no enforcement, git merge is the arbiter

---

## Project status

OpenTasks is early and intentionally small. The format is still allowed to evolve, but changes should preserve the core promise: plain markdown files, stable task/question IDs, and no required service or runtime. See [CHANGELOG.md](CHANGELOG.md) for notable changes.

## Contributing

Contributions are welcome when they keep the convention lightweight. Start with [CONTRIBUTING.md](CONTRIBUTING.md), follow the [Code of Conduct](CODE_OF_CONDUCT.md), and see [SECURITY.md](SECURITY.md) for vulnerability reporting.

## License

MIT. See [LICENSE](LICENSE).

---

## Installation

### Codex

```bash
git clone https://github.com/luisalima/opentasks-skill ~/.codex/skills/opentasks
```

Restart Codex after installing so the skill loader picks it up.

### Claude Code global

```bash
git clone https://github.com/luisalima/opentasks-skill ~/.claude/skills/opentasks
```

### Claude Code project-level

```bash
git clone https://github.com/luisalima/opentasks-skill .claude/skills/opentasks
```

Then invoke with `/opentasks`.

The repository root contains `SKILL.md`, so the clone target directory name (`opentasks`) is the command name.

### Updating

Each install is a git clone — update it with `git pull`:

```bash
git -C ~/.claude/skills/opentasks pull    # Claude Code global
git -C ~/.codex/skills/opentasks pull     # Codex (restart Codex afterwards)
git -C .claude/skills/opentasks pull      # Claude Code project-level
```

Check which version is installed with `git -C ~/.claude/skills/opentasks log -1`. A stale install silently applies an old convention, so update before relying on newer fields or operations. After updating, run `/opentasks migrate` in existing projects to upgrade their `docs/tasks/` folders to the current convention.

---

## Usage

```
/opentasks bootstrap              Set up docs/tasks/ in a new project
/opentasks new task <title>       Create a new task
/opentasks new question <title>   Create a new question
/opentasks start <item>           Mark a task as in progress
/opentasks block <item> [reason]  Mark an item as blocked
/opentasks done <item>            Close an item (task or question)
/opentasks reopen <item>          Reopen a closed item
/opentasks list [filter]          List items by status, deliverable, or questions
/opentasks sync                   Rebuild TASK_INDEX.md from frontmatter
/opentasks migrate                Upgrade a legacy docs/tasks/ folder in place
/opentasks status                 Show open/blocked items (default)
```

`<item>` can be a slug, filename, `T<N>` shorthand like `T4`, `Q<N>` shorthand like `Q3`, or an exact title match. Common aliases such as `close Q3`, `mark done`, `add task`, and `open question` resolve to the canonical operations.

---

## Setup

After installing the skill, run `/opentasks bootstrap` in your project. This:
1. Creates `docs/tasks/` with a `README.md` and empty `TASK_INDEX.md`.
2. Appends a standing instruction block to `AGENTS.md` (and `CLAUDE.md` if present) when that section is not already present.

The appended block looks like this:

```markdown
## Task and question tracking

This project uses `docs/tasks/` as a lightweight repo convention for work items and open decisions. Use the `/opentasks` skill to manage it.

- When planning or breaking down work, record concrete steps as tasks (`/opentasks new task <title>`) and open decisions as questions (`/opentasks new question <title>`).
- Keep tasks sized for one focused agent session or one coherent PR. Split work with multiple outputs, owners, or unresolved decisions.
- Use questions for unresolved decisions, ADRs for durable decisions, and tasks for execution; link ADR-derived tasks back to the ADR in `links:`.
- Keep status current: mark items `doing` when you start, `blocked` when waiting, `done` when complete.
- Starting a task records and announces a claim (`claimed_by: <who> @ <where>`). Claims are attribution, not locks.
- Never create task or question files manually — always go through `/opentasks` to keep the index in sync.
```

---

## How it works

### Folder layout

```
docs/tasks/
├── README.md            # Conventions for humans (generated by bootstrap)
├── TASK_INDEX.md        # Derived aggregate view
├── t<N>-<slug>.md       # One file per task, numbered
└── q<N>-<slug>.md       # One file per question, numbered
```

Flat — no subfolders, no hidden state, no required CLI. One file per item. Closed items stay as history; never delete.

### Two item types

**Tasks** track work. Frontmatter:
```yaml
---
status: todo            # todo | doing | blocked | done
type: task
id: T<N>                # stable task identifier, monotonic and never reused
deliverable: D2         # project-specific bucket
created: YYYY-MM-DD
links: []               # optional related URLs or repo paths
priority: p2            # optional: p1 | p2 | p3; treated as p2 when absent
depends_on: []          # optional list of task IDs this task waits on, e.g. [T3, T7]
started: YYYY-MM-DD     # added by start; kept on reopen
claimed_by: who @ where # added by start; attribution, not a lock
closed: YYYY-MM-DD      # only when done; removed on reopen
output: path/to/file.md # only if the task produced an artifact
---
```

**Questions** track open decisions. Frontmatter:
```yaml
---
status: todo            # todo | blocked | done  (skip "doing")
type: question
owner: <name-or-role>
created: YYYY-MM-DD
closed: YYYY-MM-DD      # only when done
---
```

### Status semantics

| Status    | Tasks                   | Questions              |
|-----------|-------------------------|------------------------|
| `todo`    | Not started             | Ready to ask           |
| `doing`   | In progress             | Not valid              |
| `blocked` | Waiting on dependency   | Waiting for an answer  |
| `done`    | Completed               | Answered               |

### Task body essentials

Task files include a `## Done when` section with concrete completion criteria. A task should only be closed when those criteria are satisfied or intentionally waived. Related issues, PRs, docs, branches, commits, or local paths can be recorded in the optional `links:` frontmatter list. An optional `priority: p1|p2|p3` field orders work — absent means `p2`. Machine-readable dependencies go in the optional `depends_on: [T3, T7]` list; a task is **ready** when it is `todo` and every dependency is `done`. Unknown IDs, self-references, and cycles are validation errors. When a task is started, `claimed_by: <who> @ <where>` records which agent or person picked it up and where it is running, and the agent announces the claim in its reply — if two checkouts claim the same task, the git merge conflict is the signal.

### Task sizing and agent behavior

A task should be small enough for one focused agent session or one coherent PR. Good tasks have one objective, concrete `Done when` criteria, independent verification, and no unresolved design choice hidden inside the scope.

Split a task when it has multiple outputs, multiple owners, unresolved decisions, or a title that naturally contains "and then." Do not create tasks for every tiny edit. Create them when work needs to survive chat context, coordinate across humans or agents, or show up in git history.

Agents should create tasks when breaking down a user-approved plan, discovering follow-up work that should not be done immediately, finding a blocker or dependency, extracting implementation work from an ADR, or leaving continuation work for another human or agent. Agents should not create tasks merely to describe work they are already completing in the same turn.

### ADRs and decision flow

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

### Index format

`TASK_INDEX.md` is a flat checklist grouped by deliverable (tasks) and owner (questions):

```markdown
> Frontmatter is the source of truth. This index is a derived view.

## D1 — <Deliverable name>

- [ ] [T1. Title](t1-slug.md) — `todo`
- [ ] [T2. Title](t2-slug.md) — `blocked` (waiting on client data)
- [x] [T3. Title](t3-slug.md) — `done` → path/to/output.md

## Open questions

**For <person>:**
- [ ] [Q1. The question?](q1-slug.md) — `todo`

**Answered (history):**
- [x] [Q2. Another question?](q2-slug.md) — `done`
```

`[x]` only when `status: done`. Everything else uses `[ ]`, including `blocked`.
Task lines include the stable task ID in the visible label, e.g. `T4. Implement cache`.
Non-default priorities appear as a tag after the status (`` `todo` `p1` ``); the default `p2` is never shown.

### Optional validation

`scripts/opentasks-lint` checks a `docs/tasks/` folder against the convention: frontmatter parses, statuses and priorities are valid, task IDs are present and unique, `depends_on` references resolve without self-references or cycles, and `TASK_INDEX.md` matches the files. It is a single Python 3 file with no dependencies — and entirely optional; the convention works without it.

```bash
scripts/opentasks-lint docs/tasks   # exits 1 with one finding per line
```

Copy it into a repo (or vendor it however you like) to run in CI. The skill never requires it.

### Naming conventions

- Task slugs: `t<N>-<slug>`, e.g. `t1-llm-shortlist`, `t8-extract-analysis-scripts` — N is monotonic, never reused
- Question slugs: `q<N>-<slug>`, e.g. `q1-classification-rubric` — N is monotonic, never reused
- All filenames: lowercase ASCII, accents stripped, spaces → hyphens
