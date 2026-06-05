# the-tasks-folder-skill

A [Claude Code](https://claude.ai/code) skill that manages the `docs/tasks/` flat file tracking system — a lightweight, file-based alternative to Jira/Linear for small and medium projects.

The system tracks both **work items** and **open questions** in a single folder using markdown + YAML frontmatter. Frontmatter is the source of truth; the index is a derived view.

---

## Installation

### Global (available in all projects)

```bash
git clone https://github.com/luisalima/the-tasks-folder-skill ~/.claude/skills/tasks
```

### Project-level (this project only)

```bash
git clone https://github.com/luisalima/the-tasks-folder-skill .claude/skills/tasks
```

Then invoke with `/tasks` in any Claude Code session.

---

## Usage

```
/tasks bootstrap              Set up docs/tasks/ in a new project
/tasks new task <title>       Create a new task
/tasks new question <title>   Create a new question
/tasks start <slug>           Mark a task as in progress
/tasks block <slug> [reason]  Mark an item as blocked
/tasks done <slug>            Close an item (task or question)
/tasks sync                   Rebuild TASK_INDEX.md from frontmatter
/tasks status                 Show open/blocked items (default)
```

---

## Folder layout

```
docs/tasks/
├── README.md            # Conventions for humans
├── TASK_INDEX.md        # Derived aggregate view
├── <slug>.md            # One file per task
└── q<N>-<slug>.md       # One file per question, numbered
```

- Flat — no subfolders.
- One markdown file per item.
- Closed items stay in the folder as history; never delete.
- Question filenames are prefixed `q<N>-` where N is a stable monotonic number.

---

## Spec

The full specification lives in [`SPEC.md`](https://github.com/luisalima/the-tasks-folder/blob/main/SPEC.md) in the companion repo.
