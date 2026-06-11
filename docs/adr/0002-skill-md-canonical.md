# ADR-0002: SKILL.md is the canonical convention definition

- **Status:** accepted
- **Date:** 2026-06-11

## Context

The convention is currently defined twice, in full: `SKILL.md` (agent instructions) and `README.md` (human documentation) both restate the frontmatter reference, status table, templates, naming rules, and index format. CONTRIBUTING makes "README and skill instructions agree" a PR checklist item, which acknowledges the problem without solving it: every convention change must land in two places, and drift is silent until someone notices a contradiction.

## Decision

`SKILL.md` is the single canonical definition of the convention — templates, frontmatter fields, status semantics, naming rules, and index format live there and only there.

`README.md` keeps what humans need to evaluate and install the project — the positioning ("what this is / is not"), installation, the usage table, and a short conceptual overview — and links into `SKILL.md` for the normative details instead of restating them.

## Consequences

- The README shrinks substantially; its remaining prose must not duplicate normative content.
- The CONTRIBUTING checklist item changes from "README and SKILL.md agree" to "normative changes land in SKILL.md; README links remain valid."
- One-time editing cost to deduplicate (task T9), repaid on every future convention change.
- Risk: people skim READMEs and not skill files; mitigated by keeping the conceptual overview and one worked example in the README, plus the `examples/` folder (T8).
