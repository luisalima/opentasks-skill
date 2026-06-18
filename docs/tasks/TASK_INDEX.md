> Frontmatter is the source of truth. This index is a derived view — if they disagree, read the individual .md files.

## D1 — Agent self-dispatch

- [x] [T1. Add `priority:` frontmatter field](t1-add-priority-field.md) — `done` → SKILL.md
- [x] [T2. Add `depends_on:` frontmatter field](t2-add-depends-on-field.md) — `done` → SKILL.md
- [x] [T3. Add `next` operation](t3-add-next-operation.md) — `done` → SKILL.md
- [x] [T4. Add `claimed_by:` attribution to `start`](t4-add-claimed-by-attribution.md) — `done` → SKILL.md
- [x] [T5. Add `graph` operation (Mermaid)](t5-add-graph-operation.md) — `done` → SKILL.md

## D2 — Lifecycle & tooling

- [x] [T6. Add `migrate` operation](t6-add-migrate-operation.md) — `done` → SKILL.md
- [x] [T7. Add optional validation script](t7-add-validation-script.md) — `done` → scripts/opentasks-lint

## D3 — Docs & examples

- [x] [T8. Add `examples/` folder with a populated sample](t8-add-examples-folder.md) — `done` → examples/docs/tasks/
- [x] [T9. Make SKILL.md canonical; slim the README](t9-canonicalize-skill-md.md) — `done` → README.md
- [x] [T10. Document how to update an installed skill](t10-document-skill-updates.md) — `done` → README.md

## Open questions

**For luisa:**
- [ ] [Q1. Should task state live on a branch that is always pushed, and how?](q1-tasks-on-always-pushed-branch.md) — `todo`

## Dependency graph

```mermaid
graph TD
    T1["T1. Add priority: frontmatter field"]:::done
    T2["T2. Add depends_on: frontmatter field"]:::done
    T3["T3. Add next operation"]:::done
    T5["T5. Add graph operation (Mermaid)"]:::done
    T1 --> T3
    T2 --> T3
    T2 --> T5
    classDef todo fill:#ffffff,stroke:#495057
    classDef doing fill:#fff3bf,stroke:#f59f00,stroke-width:2px
    classDef blocked fill:#ffe3e3,stroke:#e03131,stroke-dasharray: 5 5
    classDef done fill:#e9ecef,stroke:#adb5bd,color:#868e96
    classDef missing fill:#ffffff,stroke:#e03131,stroke-dasharray: 2 2,color:#e03131
```
