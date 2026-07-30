---
task_id: functional-erp-audit
priority: 8
findings_folder: findings/functional-erp/
status: pending
---

# Task 8: Functional ERP audit

## Why this task exists
This is the third of four independent, parallel audits (tasks 6-9) forming the second,
consulting-style audit pass over VERIDIAN. This task is the **functional completeness**
workstream, run blind to tasks 6 and 7 (reconciliation happens only in task 10). The Owner gave
explicit authority to edit, tighten, and rework this audit's scope -- restating the framework back
is not an acceptable answer to any question below.

## Who you are for this task
You are evaluating VERIDIAN adopting each of these perspectives in turn: an **ERP/SAP/Oracle/MS
Dynamics implementation consultant**, a **Product Manager**, and a **QA Director**. Your job is to
be exhaustive about functional coverage within each module you examine -- every module, function,
input, output, business rule, exception, report, dashboard, workflow, notification, validation,
calculation, integration, approval, and dependency -- not to give a one-line impression per module.

## Scale reality -- read this before starting
`FChecklist/compliance-tracker` (public repo) has, as of a same-day count on 2026-07-30, over 100
route folders under `src/app/(app)/` and a dedicated ERP sub-area at `src/app/(app)/erp/` with (real,
checked 2026-07-30) at least: `bank-reconciliation`, `budgets`, `cash-management`, `clm-library`,
`contracts`, `credit-notes`, `customers`, `fixed-assets`, `goods-receipt`, `inventory`,
`inventory-planning`, `invoicing`, `journal-entries`, `payment-entries`, `payroll`, `periods`,
`procurement`, `reports`, `returns`, `suppliers`. A genuinely exhaustive audit of all 100+ modules
at full depth is not realistic in one pass. Do not fake exhaustiveness by giving every module a
shallow one-line verdict -- instead, use the two-tier method below, and be honest in your findings
about which modules got deep coverage and which got only the inventory-level pass.

## Required method: two-tier coverage, not uniform shallow coverage

### Tier 1 -- Full inventory (breadth, shallow)
Re-list `src/app/(app)/` and `src/app/(app)/erp/` yourself (do not trust the list above as final --
confirm it's still current). Produce a complete table of every real module found, with one column
each for: does it have a real UI, does it have a backing API route you could find, does it have a
service file, and a one-line note on whether it looks populated/real vs. a stub/placeholder page.
This tier's job is coverage, not depth.

### Tier 2 -- Deep functional trace (depth, on a deliberately chosen sample)
Pick **8 modules** spanning both construction/PM and ERP/finance, at minimum including:
`procurement` (or `erp/procurement`), `invoicing`, `payroll`, `site-diary`, `rfis`,
`change-orders`, `approvals`, and `reports` -- plus 1-2 more of your choosing that Tier 1 flagged as
looking substantial. For **each** of the 8, answer all of the following with real evidence (an
opened file, a traced code path, a real input/output you confirmed):

1. **Every function within the module**: what are the real, distinct operations a user can perform
   (create, edit, approve, cancel, void, export, etc.)? List them by name.
2. **Inputs**: for the primary create/edit operation, what fields does it actually accept, and are
   they validated (client-side, server-side, both, or not at all)?
3. **Outputs**: what does a successful operation actually produce (a DB row, a generated document,
   a notification, an audit entry)? Confirm at least one concretely.
4. **Business rules**: name at least one real business rule enforced in code (e.g. an amount
   threshold, a status-transition guard) -- quote the actual condition or function.
5. **Exceptions/edge cases**: what happens on invalid input, a duplicate, or an out-of-order action
   (e.g. approving something already rejected)? Is this handled, or does it fail silently /
   ungracefully?
6. **Reports**: does this module have a real, working report (not just a route that returns an
   empty or placeholder result)? Name it.
7. **Dashboard**: is this module's data reflected in any real dashboard
   (`src/app/(app)/dashboard`, `construction-dashboard`, `kpi-hub`)?
8. **Workflow**: does this module have a real multi-step workflow/state machine (e.g. draft ->
   submitted -> approved -> executed), or is it single-step CRUD?
9. **Notifications**: does completing an action in this module trigger any real notification
   (email, in-app, `src/app/api/notifications`, `src/lib/loop-insight-notifier.ts`)? Confirm, don't
   assume.
10. **Validation**: summarized from #2 -- is validation consistent between client and server, or
    does the server trust client-side-only checks (a real security/data-integrity gap if so)?
11. **Calculation**: if the module involves a real calculation (e.g. payroll totals, invoice tax,
    procurement cost rollups), find the actual calculation code and confirm it does what it claims.
12. **Integration**: does this module integrate with any other real module or external system (e.g.
    procurement -> inventory -> goods-receipt -> invoicing chain)? Trace one real cross-module call.
13. **Approval**: if applicable, is there a real approval step, and does it match what an ERP
    consultant would expect (role-based, amount-based, or sequential escalation)?
14. **Dependency**: what does this module depend on that, if missing/broken, would break it (a
    specific service, a specific table)?

## What you are actually being asked to determine (summary verdicts)
- For the full Tier 1 inventory: how many modules look real/populated vs. stub/placeholder?
- For each of the 8 Tier 2 modules: a clear completeness verdict -- **complete** (all 14 dimensions
  have a real answer), **partial** (name which dimensions are missing), or **thin/stub** (mostly
  UI shell with no real logic behind it).
- Across all 8, what is the single most common missing dimension (e.g. "reports exist for 2 of 8,
  notifications exist for 1 of 8")? This kind of cross-module pattern is exactly what task 10's
  root-cause synthesis will need -- report the raw pattern here, do not cluster it into a root
  cause yourself (that's task 10's job, done only after reading all 4 of tasks 6-9's findings).

## What NOT to do
- Do not give every module in Tier 1 a Tier-2-depth answer -- that produces shallow answers for
  all 100+ and defeats the point of choosing a real sample.
- Do not give Tier 1 module names without having actually opened at least the route file or page
  component to check it's not a completely empty stub.
- Do not restate a module's name and assume its function is what the name implies -- confirm with
  real code, exactly as tasks 4 and 5 in this repo require for their pointers.
- Do not pad findings with generic "every ERP needs X" commentary not grounded in something you
  actually checked here.

## Required steps (3-pass methodology from the repo README, adapted)
1. Run the full Tier 1 inventory and Tier 2 deep-trace analysis **three independent times** for the
   8 chosen modules (you may reuse the same 8-module selection across passes for consistency, or
   vary it -- state which you did). Re-derive each pass fresh.
2. Produce **one** distilled, reconciled analysis: one inventory table, one verdict per Tier-2
   module, resolving any disagreement between passes with evidence.
3. Identify concrete, named gaps per module.
4. Give close-ended remediation measures only -- e.g. "add a server-side amount-threshold check in
   the purchase-order approval handler mirroring the client-side check already present in
   `<file>`," not "improve validation."

## Output
Write your findings to `findings/functional-erp/` (see that folder's `README.md` for the exact
expected shape). End with `STATUS: functional-erp-audit COMPLETE` or
`STATUS: functional-erp-audit BLOCKED -- <one-sentence reason>`.
