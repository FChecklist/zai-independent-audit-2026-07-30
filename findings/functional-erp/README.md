# findings/functional-erp/

Output location for `tasks/8-functional-erp-audit.md`.

## Expected shape
Create one new file here (e.g. `functional-erp-audit-2026-07-30.md`) via GitHub's web "Add file"
button, containing:

```markdown
# Functional ERP audit -- <date>

## Tier 1 inventory
<complete table of every real module found under src/app/(app)/ and src/app/(app)/erp/, with
columns for real UI / real API / real service / populated-vs-stub note>

## Pass 1 (Tier 2 deep trace)
<full independent 14-dimension trace for each of the 8 chosen modules, with real evidence>

## Pass 2 (Tier 2 deep trace)
<full independent analysis, re-derived, not copy-pasted from pass 1>

## Pass 3 (Tier 2 deep trace)
<full independent analysis, re-derived>

## Reconciled analysis
<the ONE distilled, deterministic reconciliation of the three Tier-2 passes -- one completeness
verdict (complete / partial / thin-stub) per module, resolved with evidence; plus the
most-common-missing-dimension pattern across the 8 modules, reported raw (not clustered into a
root cause -- that is task 10's job)>

## Gaps identified
<concrete, named, evidenced gaps per module>

## Recommended remediation (close-ended only)
<specific, actionable items -- named file/check/field to add, not vague advice>

STATUS: functional-erp-audit COMPLETE
```

If you could not complete the task, a shorter file ending in
`STATUS: functional-erp-audit BLOCKED -- <one-sentence reason>` is a valid and useful output --
do not force a COMPLETE you didn't earn.
