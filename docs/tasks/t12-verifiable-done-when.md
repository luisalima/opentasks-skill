---
status: todo
type: task
id: T12
deliverable: D1
created: 2026-06-20
links:
  - docs/feedback/multi-agent-automation-feedback.md
priority: p1
---

# T12. Make `Done when` independently verifiable; add optional `verify:` field

## Objective
An automation gate has to confirm success without trusting the implementer. Vague `Done when` ("implement X") blocks unattended execution, and for security tasks the happy path is not enough — the negative/adversarial case must be asserted. This was the single most important readiness signal in the automation pilot (feedback #4).

## What we need to extract / do
- Strengthen the task template and guidance in `SKILL.md` (and the README): `Done when` must be concrete and **independently checkable by whatever means the repo supports** — not a restatement of the work. The *form* of the check is repo-dependent: a test command where a test harness exists, a lint/validation run (e.g. this repo's `opentasks-lint`), an assertable contract, or an observable artifact/behavior otherwise. Do not mandate a universal mechanism (e.g. "must be a unit test") — a docs-only or convention repo verifies differently than an app.
- Add explicit guidance: for security-relevant tasks, `Done when` must include the adversarial/negative assertion (rejected input, blocked request), not only the success path.
- Add an **optional** `verify:` frontmatter line to the reference and template: how to verify the task independently (e.g. a command to run). It is a convenience pointer, not a required field — absent is fine, and what goes in it depends on the repo.
- Update the §README required-content checklist so the README mirrors the strengthened guidance and the optional field.

## Done when
- `SKILL.md` template, frontmatter reference, and §README guidance require a concrete, independently-checkable `Done when` (mechanism left to the repo) and document the optional `verify:` field.
- The guidance explicitly states the verification form is repo-dependent and does not mandate a single mechanism.
- The security guidance explicitly requires an adversarial/negative assertion for security-relevant tasks.

## Output
Updated `SKILL.md` and `README.md`.

## Dependencies
None.
