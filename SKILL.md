---
name: opentasks
description: "Manage a docs/tasks/ flat-file tracking system: bootstrap the folder, create tasks/questions, update status, close or reopen items, list open work, and rebuild the index."
when_to_use: "Use when the user asks to set up a tasks folder, create a task, open a question, close or mark done, mark blocked/doing, reopen an item, show open tasks, list questions, or sync a task index. Trigger phrases include: create a task for X; open a question about Y; close Q3; mark done; set up the tasks folder; show open tasks; add a question about Y."
argument-hint: "[bootstrap | new task <title> | new question <title> | start <item> | block <item> [reason] | done <item> | reopen <item> | list [filter] | sync | status]"
---

# /opentasks — docs/tasks/ folder manager

You are managing the `docs/tasks/` flat file tracking system. The argument you received is:

**$ARGUMENTS**

Parse the intent and execute the appropriate operation below. If the argument is empty or ambiguous, run **status**.

Accept common aliases:
- `init` → `bootstrap`
- `create task`, `add task`, `task` → `new task`
- `open question`, `add question`, `question` → `new question`
- `close`, `finish`, `complete`, `mark done` → `done`

Where an operation accepts `<item>`, resolve it before editing:
1. Exact filename under `docs/tasks/`, with optional `.md`.
2. Exact slug, meaning filename without `.md`.
3. Question shorthand `Q<N>` / `q<N>` → the unique `q<N>-*.md` file.
4. Exact case-insensitive title match from the file heading or index line.

If resolution finds no item, stop and tell the user what was searched. If it finds multiple items, stop and ask for the exact slug.

---

## Folder guard

Before executing any operation other than `bootstrap` / `init`, check whether `docs/tasks/` exists. If it does not, stop and tell the user:

> No `docs/tasks/` folder found. Run `/opentasks bootstrap` to set up the system first.

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

This project uses `docs/tasks/` to track work items and open decisions. Use the `/opentasks` skill to manage it.

- When planning or breaking down work, record concrete steps as tasks (`/opentasks new task <title>`) and open decisions as questions (`/opentasks new question <title>`).
- Keep status current: mark items `doing` when you start, `blocked` when waiting, `done` when complete.
- Never create task or question files manually — always go through `/opentasks` to keep the index in sync.
```

---

### `new task <title>`
Create a new task file.
1. Generate a kebab-case slug from the title (ASCII, lowercase, hyphens; strip accents, replace spaces with hyphens).
2. If `docs/tasks/<slug>.md` already exists, abort and tell the user the file exists — suggest they pick a different title or run `/opentasks status` to see the existing item.
3. Determine the deliverable bucket from context or ask the user.
4. Write `docs/tasks/<slug>.md` with this exact structure:

```markdown
---
status: todo
type: task
deliverable: <D1|D2|ops|…>
created: <YYYY-MM-DD>
---

# <Title>

## Objective
<One or two sentences — what this is and why it matters.>

## What we need to extract / do
<Concrete bullets describing the actual work.>

## Output
<What gets produced and where it feeds into. Write "none" if the task produces no tracked artifact.>

## Dependencies
<What must exist first.>
```

5. Add a line to `TASK_INDEX.md` under the correct `## <Deliverable>` group:
   `- [ ] [<Title>](<slug>.md) — \`todo\``

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

### `start <item>`
1. Resolve `<item>` and open the matching file.
2. If the file has `type: question` in frontmatter, abort and tell the user: questions don't use `doing` status — use `/opentasks block` to flag as waiting, or `/opentasks done` to record the answer.
3. If `status: done`, abort and tell the user to run `/opentasks reopen <item>` first.
4. Change `status: todo` or `status: blocked` → `status: doing` in frontmatter. If it is already `doing`, leave it as-is.
5. Add `started: <YYYY-MM-DD>` to frontmatter directly after `status` if absent. Do not duplicate or overwrite an existing `started:`.
6. If a `## Blocker` section is present and the blocker is no longer relevant, remove it or add a short resolved note.
7. Update the index line: replace the current status tag with `` `doing` `` and remove any `(waiting on ...)` suffix.

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
3. Otherwise set `status: done`, add `closed: <YYYY-MM-DD>` to frontmatter if absent, and do not duplicate an existing `closed:`.
4. **If it's a question:** add the answer inline in the body with date and source (do NOT replace the question text). If the answer is not clear from context, ask for it before closing.
5. **If it's a task that produced a tracked artifact:** add or update `output: <path>` in frontmatter. Omit `output:` entirely if the task produced no tracked artifact.
6. In `TASK_INDEX.md`:
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

Output format: same grouped list style as the index — one item per line with checkbox and status tag. Print a count summary at the bottom.

---

### `sync`
Rebuild `TASK_INDEX.md` from scratch by reading all files in `docs/tasks/`.
1. Read frontmatter from every `*.md` file except `README.md` and `TASK_INDEX.md`.
2. Group `type: task` files by `deliverable`; group `type: question` files by `owner`.
3. For backward compatibility, treat files with no `type` and a `deliverable` as tasks, but report them as frontmatter mismatches.
4. Emit the full index in the format defined in **§INDEX**.
5. Frontmatter is the source of truth — discard the old index content entirely.

---

### `status` (default when no argument given)
Read `TASK_INDEX.md` (or scan `docs/tasks/` if the index is absent) and report:
- Count of items by status: todo / doing / blocked / done.
- All `doing` and `blocked` items with their blockers.
- Any frontmatter/index mismatches spotted, including missing `type`, invalid statuses, questions marked `doing`, tasks missing `deliverable`, questions missing `owner`, duplicate question numbers, and stale index lines.

---

## File naming rules

- Task slugs: short, descriptive kebab-case. Examples: `llm-shortlist`, `extract-analysis-scripts`.
- Question slugs: `q<N>-<short-slug>`. Examples: `q1-classification-rubric`, `q7-template-wording`.
- All filenames are lowercase ASCII — strip accents, replace spaces with hyphens.
- Question numbers are monotonic across the project lifetime. Never reuse a number.

---

## Status semantics

| Status    | Tasks                        | Questions                      |
|-----------|------------------------------|--------------------------------|
| `todo`    | Not started                  | Ready to ask / discuss         |
| `doing`   | In progress                  | Not valid — use `block` or `done` directly |
| `blocked` | Waiting on dependency        | Waiting for an answer          |
| `done`    | Completed                    | Answered                       |

---

## Frontmatter reference

**Tasks:**
```yaml
---
status: todo            # todo | doing | blocked | done
type: task
deliverable: D2         # project-specific bucket
created: YYYY-MM-DD
started: YYYY-MM-DD     # added by `start`; kept on reopen as historical record
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

1. One-paragraph intro: this folder tracks both execution tasks and open questions using a flat markdown + YAML frontmatter convention; item type is distinguished by the `type` frontmatter field.
2. How it works: one file per item; frontmatter is the source of truth; `TASK_INDEX.md` is a derived view.
3. The four status values and what they mean for tasks vs questions (see table above).
4. Type conventions: task vs question; what `owner` is for.
5. The full task body template as a fenced markdown block.
6. The full question body template as a fenced markdown block.
7. Workflow: create → start → block → close → reopen, with the exact frontmatter changes at each step.
8. A note that closed files are kept as history — never delete.

---

## §INDEX — `TASK_INDEX.md` format

```markdown
> Frontmatter is the source of truth. This index is a derived view — if they disagree, read the individual .md files.

## D1 — <Deliverable name>

- [ ] [<Title>](<slug>.md) — `todo`
- [ ] [<Title>](<slug>.md) — `blocked` (waiting on client data)
- [x] [<Title>](<slug>.md) — `done` → path/to/output.md

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
- `blocked` items may have a parenthetical explanation: `(waiting on client data)`.
- Append `→ path/to/output` for `done` tasks that produced a tracked artifact.
- Closed files are never deleted — they remain in the folder and in the index as history.
