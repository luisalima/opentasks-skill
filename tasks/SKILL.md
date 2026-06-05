---
description: Manage the docs/tasks/ flat file tracking system — bootstrap the folder, create tasks/questions, update status, close items, sync the index. Trigger phrases: "create a task for X", "open a question about Y", "close Q3", "mark done", "set up the tasks folder", "show open tasks", "add a question about Y".
argument-hint: [bootstrap | new task <title> | new question <title> | start <slug> | block <slug> | done <slug> | sync | status]
---

# /tasks — docs/tasks/ folder manager

You are managing the `docs/tasks/` flat file tracking system. The argument you received is:

**$ARGUMENTS**

Parse the intent and execute the appropriate operation below. If the argument is empty or ambiguous, run **status**.

---

## Operations

### `bootstrap` / `init`
Set up a fresh `docs/tasks/` folder in a new project.
1. Ask the user for the project's deliverable names if not obvious from context (e.g. D1, D2, ops).
2. Confirm what language the project documentation is in (default to English if obvious).
3. Create `docs/tasks/README.md` — a human-readable conventions doc. See **§README** below for required content.
4. Create `docs/tasks/TASK_INDEX.md` — empty index with group headers for each deliverable + Open questions section. See **§INDEX** below for format.
5. Do NOT create any tasks or questions yet.

---

### `new task <title>`
Create a new task file.
1. Generate a kebab-case slug from the title (ASCII, lowercase, hyphens; strip accents, replace spaces with hyphens).
2. Determine the deliverable bucket from context or ask the user.
3. Write `docs/tasks/<slug>.md` with this exact structure:

```markdown
---
status: todo
deliverable: <D1|D2|ops|…>
created: <YYYY-MM-DD>
---

# <Title>

## Objective
<One or two sentences — what this is and why it matters.>

## What we need to extract / do
<Concrete bullets describing the actual work.>

## Output
<What gets produced and where it feeds into.>

## Dependencies
<What must exist first.>
```

4. Add a line to `TASK_INDEX.md` under the correct `## <Deliverable>` group:
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

### `start <slug>`
1. Open `docs/tasks/<slug>.md`.
2. Change `status: todo` → `status: doing` in frontmatter.
3. Update the index line: replace the current status tag with `` `doing` ``.

---

### `block <slug> [reason]`
1. Open `docs/tasks/<slug>.md`.
2. Change status → `status: blocked` in frontmatter.
3. Add or update a `## Blocker` section in the body explaining what's being waited on.
4. Update the index line: replace the current status tag with `` `blocked` `` and append `(waiting on <reason>)` if a reason was given.

---

### `done <slug>`
1. Open `docs/tasks/<slug>.md`.
2. Set `status: done`, add `closed: <YYYY-MM-DD>` to frontmatter.
3. **If it's a question:** add the answer inline in the body with date and source (do NOT replace the question text).
4. **If it's a task with a tracked output:** add `output: <path>` to frontmatter.
5. In `TASK_INDEX.md`:
   - Flip `[ ]` → `[x]` on the item's line.
   - Replace status tag with `` `done` ``.
   - For tasks: append `→ <output-path>` if there is an output.
   - For questions: move the line to the `**Answered (history):**` sub-group under `## Open questions`.

---

### `sync`
Rebuild `TASK_INDEX.md` from scratch by reading all files in `docs/tasks/`.
1. Read frontmatter from every `*.md` file except `README.md` and `TASK_INDEX.md`.
2. Group tasks by `deliverable`; group questions by `owner`.
3. Emit the full index in the format defined in **§INDEX**.
4. Frontmatter is the source of truth — discard the old index content entirely.

---

### `status` (default when no argument given)
Read `TASK_INDEX.md` (or scan `docs/tasks/` if the index is absent) and report:
- Count of items by status: todo / doing / blocked / done.
- All `doing` and `blocked` items with their blockers.
- Any frontmatter/index mismatches spotted.

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
| `doing`   | In progress                  | (skip — use blocked or done)   |
| `blocked` | Waiting on dependency        | Waiting for an answer          |
| `done`    | Completed                    | Answered                       |

---

## Frontmatter reference

**Tasks:**
```yaml
---
status: todo            # todo | doing | blocked | done
deliverable: D2         # project-specific bucket
created: YYYY-MM-DD
closed: YYYY-MM-DD      # only when status = done
output: path/to/file.md # only if the task produced a tracked artifact
---
```

**Questions:**
```yaml
---
status: todo            # todo | doing | blocked | done
type: question
owner: <name-or-role>
created: YYYY-MM-DD
closed: YYYY-MM-DD      # only when status = done
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
7. Workflow: create → start → block → close, with the exact frontmatter changes at each step.
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
