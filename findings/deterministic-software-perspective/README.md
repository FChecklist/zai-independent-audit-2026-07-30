# findings/deterministic-software-perspective/

Output location for `tasks/2-deterministic-software-perspective-audit.md`.

## Expected shape
Create one new file here (e.g. `deterministic-software-audit-2026-07-30.md`) via GitHub's web
"Add file" button, containing:

```markdown
# Deterministic-software-perspective audit -- <date>

## Pass 1
<full independent trace-and-verify analysis, with quoted file paths/line numbers/conditions>

## Pass 2
<full independent analysis, re-derived>

## Pass 3
<full independent analysis, re-derived>

## Reconciled analysis
<the ONE distilled, deterministic reconciliation -- for each AI-invoking code path examined, a
clear verdict: gated-and-verified / gated-but-weak / ungated, with evidence>

## Access gaps encountered
<e.g. could not read gateway.py in claude-control (private) -- explicitly listed, not glossed over>

## Gaps identified
<concrete, named, evidenced gaps between the stated principle and actual enforcement>

## Recommended remediation (close-ended only)
<specific, actionable items -- named file/function/test, not vague advice>

STATUS: deterministic-software-perspective-audit COMPLETE
```

If you could not complete the task, a shorter file ending in
`STATUS: deterministic-software-perspective-audit BLOCKED -- <one-sentence reason>` is a valid and
useful output -- do not force a COMPLETE you didn't earn.
