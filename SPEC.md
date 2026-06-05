# Spec — `docs/tasks/` folder for project tracking

A flat, file-based system for tracking both **work items** and **open questions** in a single folder, using markdown + YAML frontmatter. Designed for small/medium projects where a Linear/Jira board would be overkill but informal TODOs are not enough.

Reproduce the structure exactly as described below. Write all user-facing text (README, index, task bodies) in the same language as the rest of the project's documentation — do not default to English if the project uses another language.

---

## 1. Folder layout

```
docs/tasks/
├── README.md            # conventions (this is the spec for humans)
├── TASK_INDEX.md        # derived aggregate view
├── <slug>.md            # one file per task
├── <slug>.md
└── q<N>-<slug>.md       # one file per question, numbered
```

- Flat — no subfolders.
- One markdown file per item.
- Closed items stay in the folder as history; never delete.
- Filename is a kebab-case slug. For questions, prefix with `q<N>-` where N is a stable monotonic number (`q1-...`, `q2-...`).

---

## 2. Frontmatter — the source of truth

The YAML frontmatter at the top of each file is **authoritative** for status. The index is a derived view; if they ever disagree, frontmatter wins.

### 2.1 Task frontmatter

```yaml
---
status: todo            # todo | doing | blocked | done
deliverable: D2         # project-specific bucket (D1, D2, ops, etc.)
created: YYYY-MM-DD
closed: YYYY-MM-DD      # only when status = done
output: path/to/file.md # only if the task produced a tracked artifact
---
```

`type` is omitted for tasks (or explicitly `type: task`).

### 2.2 Question frontmatter

```yaml
---
status: todo            # todo | doing | blocked | done
type: question
owner: <name-or-role>   # who needs to answer (e.g. a person, "internal")
created: YYYY-MM-DD
closed: YYYY-MM-DD      # only when status = done
---
```

### 2.3 Status semantics

- `todo` — for tasks: not started. For questions: ready to ask/discuss.
- `doing` — in progress.
- `blocked` — waiting (on client, dependency, decision). For questions, `blocked` = waiting for an answer.
- `done` — completed / answered.

---

## 3. Body structure

### 3.1 Task body template

```markdown
# <Task title>

## Objective
One or two sentences — what this is and why it matters.

## What we need to extract / do
Concrete bullets describing the actual work.

## Output
What gets produced (document, schema, decision recorded as ADR, etc.)
and where it feeds into (which deliverable / file).

## Dependencies
What must exist first (client data, another task, a decision, etc.)
```

Optional sections that can appear when useful: `## Context` (background needed to understand the task), `## Notes` (loose observations, follow-ups).

### 3.2 Question body template

```markdown
# Q<N>. <The question, phrased as a question>

**Why it matters:** impact of the answer on design / scope.
- Branch A → consequence A.
- Branch B → consequence B.

[Optional hint, prior evidence, or context.]

[**Answer** (when answered): include the date and source, e.g. "(Filipe, 2026-04-16)".]

**Still open:** what remains to clarify, even if partially answered.
```

When a question gets answered:
1. Add the answer inline in the body (don't replace the question).
2. Set `status: done` and add `closed: YYYY-MM-DD` to frontmatter.
3. If only partially answered, keep `status: blocked` and use the **Still open** section to list what's left.

---

## 4. `README.md` — conventions document

Lives at `docs/tasks/README.md`. Explains to humans how the system works. Must cover:

- One-paragraph intro: this folder tracks both execution tasks and open questions in the same convention; type is distinguished by the `type` frontmatter field.
- How it works: one file per item; frontmatter is source of truth; index is a derived view.
- Status conventions (the four values and what they mean for each type).
- Type conventions (task vs. question; what `owner` is for).
- The full task body template (as a fenced markdown block).
- The full question body template (as a fenced markdown block).
- Workflow: create → start → block → close, with the exact frontmatter changes at each step.
- Note that closed files are kept as history.

---

## 5. `TASK_INDEX.md` — derived view

A flat checklist of every item, grouped for scanability. Must include a header noting that frontmatter is the source of truth.

### Grouping

- Tasks: group by `deliverable` (e.g. `## D1 — <name>`, `## D2 — <name>`, `## Ops`).
- Questions: group under `## Open questions`, then sub-group by `owner` (e.g. `**For <person A>:**`, `**For <person B>:**`).
- A final `**Answered (history):**` sub-group for `done` questions.

### Line format

```markdown
- [ ] [<Title>](<filename>.md) — `<status>` (optional one-line note)
- [x] [<Title>](<filename>.md) — `done` → `path/to/output.md`
```

- `[x]` only when status is `done`. Everything else uses `[ ]` (including `blocked` — the box reflects whether it's closed, not whether it's actively being worked on).
- Append `→ path/to/output` for `done` tasks that produced an artifact (matches the `output:` frontmatter).
- Brief parenthetical notes are allowed for `blocked` items to explain what they're waiting on.

---

## 6. Workflow rules (for the agent maintaining the folder)

1. **Create**: new file with `status: todo` frontmatter + add a line to `TASK_INDEX.md` under the right group.
2. **Start a task**: change frontmatter to `status: doing`. (Questions skip this state.)
3. **Block**: `status: blocked` + a brief note in the body (and optionally on the index line) explaining the blocker.
4. **Close**:
   - Frontmatter → `status: done`, add `closed: YYYY-MM-DD`.
   - Tasks: if there's a tracked output, add `output: <path>` to frontmatter and `→ <path>` to the index line.
   - Questions: write the answer inline in the body with date and source.
   - Index: flip `[ ]` to `[x]`. For questions, move the line to the `Answered (history)` sub-group.
5. **Never delete** a closed file. The folder is also the project's audit trail.
6. **Keep frontmatter and index in sync**, but if they ever diverge, treat frontmatter as truth and fix the index.

---

## 7. Naming conventions

- Task slugs: short, descriptive, kebab-case. Examples: `llm-shortlist.md`, `extract-analysis-scripts.md`.
- Question slugs: `q<N>-<short-slug>.md` where N is the question's stable number. Examples: `q1-classification-rubric.md`, `q7-template-wording.md`. Numbering is monotonic across the project — never reused.
- All filenames are lowercase ASCII (kebab-case). Accents are stripped; spaces become hyphens.

---

## 8. What to do when instructed to "create the structure"

When asked to bootstrap this in a new project:

1. Create `docs/tasks/`.
2. Write `docs/tasks/README.md` containing the conventions described in §4 (in the project's documentation language).
3. Write `docs/tasks/TASK_INDEX.md` with the header note from §5 and empty group sections matching the project's deliverables (ask the user for deliverable names if not obvious).
4. Do **not** invent placeholder tasks or questions — leave the index empty under each group and let the user populate it.
