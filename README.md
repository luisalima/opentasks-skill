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
/opentasks start <item>           Claim a task and begin working on it
/opentasks claim <item>           Claim a task without starting the work
/opentasks block <item> [reason]  Mark an item as blocked
/opentasks done <item>            Close an item (task or question)
/opentasks reopen <item>          Reopen a closed item
/opentasks list [filter]          List items by status, deliverable, or questions
/opentasks next [deliverable]     Recommend the next ready task to pick up
/opentasks graph [write]          Render the depends_on graph as Mermaid (write embeds it in TASK_INDEX.md)
/opentasks sync                   Rebuild TASK_INDEX.md from frontmatter
/opentasks migrate                Upgrade a legacy docs/tasks/ folder in place
/opentasks status                 Show open/blocked items (default)
```

`<item>` can be a slug, filename, `T<N>` shorthand like `T4`, `Q<N>` shorthand like `Q3`, or an exact title match. Common aliases such as `close Q3`, `mark done`, `add task`, and `open question` resolve to the canonical operations.

---

## Setup

After installing the skill, run `/opentasks bootstrap` in your project. This creates `docs/tasks/` with a conventions `README.md` and an empty `TASK_INDEX.md`, and appends a standing "Task and question tracking" instruction block to `AGENTS.md` (and `CLAUDE.md` if present) so agents pick up the convention without being told each session. The exact block is defined in [SKILL.md › `bootstrap`](SKILL.md#bootstrap--init).

---

## How it works

[`SKILL.md`](SKILL.md) is the canonical definition of the convention — operations, templates, frontmatter fields, status semantics, naming rules, and the index format live there and only there (see [ADR-0002](docs/adr/0002-skill-md-canonical.md)). What follows is the conceptual overview.

### Folder layout

```
docs/tasks/
├── README.md            # Conventions for humans (generated by bootstrap)
├── TASK_INDEX.md        # Derived aggregate view
├── t<N>-<slug>.md       # One file per task, numbered
└── q<N>-<slug>.md       # One file per question, numbered
```

Flat — no subfolders, no hidden state, no required CLI. One file per item. Closed items stay as history; never delete. Numbers are monotonic and never reused ([naming rules](SKILL.md#file-naming-rules)).

### Items, statuses, and the index

**Tasks** track work and carry a stable `T<N>` identifier; **questions** track open decisions and carry an `owner` who needs to answer. Each file's YAML frontmatter is the source of truth ([frontmatter reference](SKILL.md#frontmatter-reference)); `TASK_INDEX.md` is a derived checklist that `sync` rebuilds from scratch ([index format](SKILL.md#index--task_indexmd-format)).

Items move `todo` → `doing` → `done`, with `blocked` for anything waiting; questions skip `doing` ([status semantics](SKILL.md#status-semantics)). Tasks close only when their `## Done when` criteria are satisfied or intentionally waived.

Optional fields order and connect the work: `priority` (`p1`/`p2`/`p3`, absent means `p2`), `links` for related ADRs, PRs, or paths, `depends_on` for machine-readable dependencies, and `verify` for an independent completion check — a task is **ready** when it is `todo` and every dependency is `done`, and `/opentasks next` recommends the highest-priority ready task ([dependencies and readiness](SKILL.md#dependencies-and-readiness)). Starting a task records `claimed_by: <who> @ <where>` — attribution, not a lock; if two checkouts claim the same task, the git merge conflict is the signal.

### A worked example

A populated folder — every status, an answered question, priorities, dependencies, a claim, and a linked ADR — lives in [`examples/docs/tasks/`](examples/docs/tasks/). One in-progress task from it:

```markdown
---
status: doing
type: task
id: T2
deliverable: D1
created: 2026-05-22
links: []
depends_on: [T1]
started: 2026-06-02
claimed_by: luisa @ herbarium-main
---

# T2. Build plant index page

## Objective
A browsable index of all plants, grouped by light needs — the main entry point of the site.

## What we need to extract / do
- Template the index page over the plant collection.
- Group entries by light requirement (full sun / partial / shade).
- Add per-plant summary cards linking to detail pages.

## Done when
- The index lists every plant in the collection, grouped, with working links.

## Output
`site/index.njk` and the card partial.
```

and its line in the derived `TASK_INDEX.md`:

```markdown
- [ ] [T2. Build plant index page](t2-build-plant-index-page.md) — `doing`
```

### Sizing, decisions, and ADRs

A task should fit one focused agent session or one coherent PR; split work with multiple outputs, owners, or unresolved decisions ([task sizing and agent behavior](SKILL.md#task-sizing-and-agent-behavior)). Its `## Done when` must be independently checkable rather than a restatement of the work — and for security-relevant tasks must assert the adversarial/negative case (input rejected, request blocked), not only the happy path. Unresolved decisions become questions, durable decisions become ADRs, and execution becomes tasks — `Q<N> → ADR → T<N>` — with ADR-derived tasks linking back via `links:` ([ADRs and decision flow](SKILL.md#adrs-and-decision-flow)).

### Optional validation

`scripts/opentasks-lint` checks a `docs/tasks/` folder against the convention: frontmatter parses, statuses and priorities are valid, task IDs are present and unique, `depends_on` references resolve without self-references or cycles, and `TASK_INDEX.md` matches the files. It is a single Python 3 file with no dependencies — and entirely optional; the convention works without it.

```bash
scripts/opentasks-lint docs/tasks   # exits 1 with one finding per line
```

Copy it into a repo (or vendor it however you like) to run in CI. The skill never requires it.
