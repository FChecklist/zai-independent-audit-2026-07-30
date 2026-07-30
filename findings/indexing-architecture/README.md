# findings/indexing-architecture/

Output location for `tasks/4-indexing-architecture-audit.md`.

## Expected shape
Create one new file here (e.g. `indexing-architecture-audit-2026-07-30.md`) via GitHub's web "Add
file" button, containing:

```markdown
# Indexing-architecture audit -- <date>

## Pass 1
<full independent analysis of all 10 index types plus the UMR verdict, with quoted file
paths/tables/commands and what you actually found when you opened/ran them>

## Pass 2
<full independent analysis, re-derived, not copy-pasted from pass 1>

## Pass 3
<full independent analysis, re-derived>

## Reconciled analysis
<the ONE distilled, deterministic reconciliation of the three passes -- for each of the 10 index
types, a clear verdict (real-and-live / real-but-partial / static-snapshot-only /
designed-but-not-built / does-not-exist) with evidence; where the three passes agree, say so
plainly; where they disagree, say so and resolve it with evidence>

## Universal Master Registry (UMR) design verdict
<the one reconciled, evidence-based verdict on the Owner's proposed UMR target architecture --
what already substantially fits (name it), what would genuinely need to change (name it), and
whether the UMR is the right target at all>

## Access notes
<e.g. repo visibility you actually observed for claude-control/projexa at the time you did this
(this task file flags a known discrepancy between this repo's main README and a same-day
re-check -- state what you found, don't just take either claim on faith)>

## Gaps identified
<concrete, named, evidenced gaps -- a specific file, table, or mechanism, not a general area>

## Recommended remediation (close-ended only)
<specific, actionable items -- named file/script/check/test, not vague advice>

STATUS: indexing-architecture-audit COMPLETE
```

If you could not complete the task, a shorter file ending in
`STATUS: indexing-architecture-audit BLOCKED -- <one-sentence reason>` is a valid and useful
output -- do not force a COMPLETE you didn't earn.
