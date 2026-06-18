# Changelog

All notable changes to this project are tracked here.

This project uses human-readable release notes.

## Unreleased

### Added

- `claim <item>` operation: record a claim (`claimed_by`, `started`, status `doing`) without beginning the work.

### Changed

- `start <item>` now claims the task **and** begins executing it in the same turn; previously it only recorded the claim.
- `SKILL.md` is the canonical convention definition (ADR-0002): the README keeps positioning, installation, usage, and a conceptual overview, and links into `SKILL.md` for templates, frontmatter, statuses, naming, and the index format.

## 0.1.0 - 2026-06-11

### Added

- MIT license.
- Stable `T<N>` task IDs and numbered task filenames.
- `links: []` frontmatter for related URLs, repo paths, and ADR references.
- `## Done when` task completion criteria.
- Task sizing guidance for agents and humans.
- ADR decision flow: `Q<N> -> ADR -> T<N>`.
- Release-readiness files for contributors, conduct, security, issues, and pull requests.
- Codex UI metadata in `agents/openai.yaml`.

### Changed

- Repositioned OpenTasks as a lightweight repo convention rather than a task manager.
- Simplified `SKILL.md` frontmatter so the skill validates cleanly.
