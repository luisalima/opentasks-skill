# Contributing

Thanks for helping improve OpenTasks.

OpenTasks is a tiny repo convention and agent skill. Contributions should keep it boring: markdown files, YAML frontmatter, stable IDs, and no required service, database, daemon, UI, or CLI.

## Good contributions

- Clarify the task/question convention.
- Improve agent instructions in `SKILL.md`.
- Tighten examples without adding much length.
- Add small validation or release docs.
- Fix README prose or links that drifted from `SKILL.md` (the canonical definition — see [ADR-0002](docs/adr/0002-skill-md-canonical.md)).

## Usually out of scope

- Project management features.
- Kanban boards, dashboards, or hosted services.
- Required CLIs or package managers.
- Calendar, CalDAV, or VTODO sync.
- Multi-agent scheduling runtimes, locking, or lease enforcement. Claim *attribution* (`claimed_by:`) is in scope; enforcing claims is not.

## Development workflow

1. Make a branch from `main`.
2. Keep changes small and focused.
3. Land normative convention changes (fields, statuses, templates, operations) in `SKILL.md` only; touch `README.md` just to keep its overview prose and links accurate.
4. Run:

```bash
git diff --check
```

5. Open a pull request that explains the convention or behavior change.

## Skill authoring guidelines

- Keep `SKILL.md` focused on instructions an agent needs while working.
- Put trigger language in the frontmatter `description`.
- Prefer examples over long explanation.
- Avoid adding helper files to the skill unless they directly improve repeated agent behavior.
- Do not make generated project files more complex than the repo convention requires.

## Pull request checklist

- The change keeps OpenTasks lightweight.
- Normative changes land in `SKILL.md`; README links into it remain valid.
- New task fields or status behavior are documented in `SKILL.md` (and reflected in the README overview only if it mentions them).
- `git diff --check` passes.
- Any change affecting existing `docs/tasks/` folders extends the `migrate` operation in the same PR, and the PR describes the impact.

## Release checklist

1. Make sure `main` is clean.
2. Update `CHANGELOG.md`.
3. Re-read `README.md` installation instructions.
4. Run `git diff --check`.
5. Create a signed or annotated tag when cutting a release.
6. Publish the GitHub release notes from the changelog entry.
