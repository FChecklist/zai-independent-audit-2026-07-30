# findings/repository-traceability/

Output location for `tasks/9-repository-traceability-audit.md`.

## Expected shape
Create one new file here (e.g. `repository-traceability-audit-2026-07-30.md`) via GitHub's web
"Add file" button, containing:

```markdown
# Repository traceability audit -- <date>

## Traceability matrix (first build)
| Business Capability | UI | API | Service | DB | Workflow | Report | Dashboard | Audit Log | Status |
|---|---|---|---|---|---|---|---|---|---|
<at least 15 rows, each cell Yes/No/Partial with a named real file/table/route as evidence>

## Verification pass
<what changed between the first build and your independent second look at the same matrix, and why>

## Final matrix
<the corrected, final version of the table above, if anything changed; otherwise state "unchanged
from first build" and why you're confident>

## Summary of broken-chain patterns
<one paragraph: which column is most often the gap (e.g. Report, Dashboard, Audit Log), and any
capability that is fully "Not Built" despite looking real in the module list>

## Gaps identified
<concrete, named, evidenced gaps -- a specific missing link in a specific chain, not a general area>

## Recommended remediation (close-ended only)
<specific, actionable items -- named file/table/route to add or connect, not vague advice>

STATUS: repository-traceability-audit COMPLETE
```

If you could not complete the task, a shorter file ending in
`STATUS: repository-traceability-audit BLOCKED -- <one-sentence reason>` is a valid and useful
output -- do not force a COMPLETE you didn't earn.

Note: this task does not use the repo's usual 3-independent-passes methodology (see the task file
for why) -- do not expect a findings file here to look like the Pass 1/2/3 shape used in the other
`findings/*/` folders.
