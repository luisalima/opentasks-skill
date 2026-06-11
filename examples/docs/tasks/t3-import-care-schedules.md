---
status: blocked
type: task
id: T3
deliverable: D1
created: 2026-05-22
links: []
depends_on: [T1]
---

# T3. Import care schedules from the nursery spreadsheet

## Objective
Watering and feeding schedules live in the nursery's spreadsheet; they need to become structured data in the plant collection.

## What we need to extract / do
- Get the export from the nursery (CSV or XLSX).
- Map columns to the plant frontmatter schema.
- Write the one-off import script and run it.

## Done when
- Every plant entry has `watering:` and `feeding:` data sourced from the spreadsheet.

## Output
Updated plant collection files plus the import script under `scripts/`.

## Dependencies
T1, and the spreadsheet export from the nursery.

## Blocker
Waiting on the nursery to send the spreadsheet export (requested 2026-06-03).
