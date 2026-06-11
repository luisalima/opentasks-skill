---
status: todo
type: task
id: T9
deliverable: D3
created: 2026-06-11
links:
  - docs/adr/0002-skill-md-canonical.md
---

# T9. Make SKILL.md canonical; slim the README

## Objective
End the full duplication of the convention between `README.md` and `SKILL.md` so future changes land in one place, per ADR-0002.

## What we need to extract / do
- Remove the normative restatements from the README: frontmatter reference, status table details, body templates, naming rules, index format.
- Keep in the README: positioning ("what this is / is not"), installation, the usage table, a short conceptual overview, and one worked example.
- Add clear links from the README into the relevant `SKILL.md` sections.
- Update the CONTRIBUTING checklist: replace "README and skill instructions agree" with "normative changes land in SKILL.md; README links remain valid."

## Done when
- No template or field reference appears in both files.
- README still lets a newcomer understand and install the project without opening SKILL.md.

## Output
Updated `README.md`, `CONTRIBUTING.md`.

## Dependencies
Best done after T1–T6 land, so the dedup isn't redone per feature; not strictly blocked.
