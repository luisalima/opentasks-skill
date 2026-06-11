---
status: done
type: question
owner: luisa
created: 2026-05-20
closed: 2026-05-21
---

# Q2. Which CSS approach — utility classes or hand-written?

**Why it matters:** affects every template written from T1 onward.
- Utility framework → faster iteration, build-step dependency.
- Hand-written → no tooling, but slower for a content-heavy site.

**Still open:** nothing.

---

**Answer (2026-05-21, decided with the nursery):** hand-written CSS. The site is small, content-first, and the no-build-tooling constraint from ADR-0001 applies to styling too.
