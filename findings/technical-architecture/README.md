# findings/technical-architecture/

Output location for `tasks/7-technical-architecture-audit.md`.

## Expected shape
Create one new file here (e.g. `technical-architecture-audit-2026-07-30.md`) via GitHub's web
"Add file" button, containing:

```markdown
# Technical architecture audit -- <date>

## Pass 1
<full independent analysis across all 13 dimensions (architecture correctness / folder structure /
maintainability / DB normalization / multi-tenant correctness / security / API correctness /
performance / metadata completeness / AI integration cleanliness / duplication / technical debt /
scalability), with real named files/tables/routes you actually opened>

## Pass 2
<full independent analysis, re-derived, not copy-pasted from pass 1, ideally sampling a different
set of files/routes/tables>

## Pass 3
<full independent analysis, re-derived>

## Reconciled analysis
<the ONE distilled, deterministic reconciliation of the three passes -- one verdict per dimension,
where the passes agree, where they disagree and why, resolved with evidence>

## Gaps identified
<concrete, named, evidenced gaps -- a specific file, table, or route, not a general area>

## Recommended remediation (close-ended only)
<specific, actionable items -- named file/check/test/CI step, not vague advice>

STATUS: technical-architecture-audit COMPLETE
```

If you could not complete the task, a shorter file ending in
`STATUS: technical-architecture-audit BLOCKED -- <one-sentence reason>` is a valid and useful
output -- do not force a COMPLETE you didn't earn.
