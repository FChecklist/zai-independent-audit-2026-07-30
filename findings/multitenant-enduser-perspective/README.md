# findings/multitenant-enduser-perspective/

Output location for `tasks/3-multitenant-enduser-perspective-audit.md`.

## Expected shape
Create one new file here (e.g. `multitenant-enduser-audit-2026-07-30.md`) via GitHub's web
"Add file" button, containing:

```markdown
# Multi-tenant end-user perspective audit -- <date>

## Pass 1
<full independent walkthrough, including literal steps taken on the live site and literal results>

## Pass 2
<full independent walkthrough, re-attempted, not copy-pasted from pass 1>

## Pass 3
<full independent walkthrough, re-attempted>

## Reconciled analysis
<the ONE distilled, deterministic reconciliation -- did isolation hold across all attempts? if
inconsistent, say so explicitly rather than averaging it away>

## Credentials / access notes
<what you were given, what you were not given, what you could not test as a result>

## Gaps identified
<concrete, named, evidenced gaps -- a specific URL/route/record, a specific broken step>

## Recommended remediation (close-ended only)
<specific, actionable items -- named route/test/check, not vague advice>

STATUS: multitenant-enduser-perspective-audit COMPLETE
```

If you could not complete the task (e.g. no demo credentials given), a shorter file ending in
`STATUS: multitenant-enduser-perspective-audit BLOCKED -- <one-sentence reason>` is a valid and
useful output -- do not force a COMPLETE you didn't earn.
