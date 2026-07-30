---
task_id: repository-traceability-audit
priority: 9
findings_folder: findings/repository-traceability/
status: pending
---

# Task 9: Repository traceability audit

## Why this task exists
This is the fourth of four independent, parallel audits (tasks 6-9) forming the second,
consulting-style audit pass over VERIDIAN, run blind to tasks 6-8 (reconciliation happens only in
task 10). This audit exists because the other three can each miss the same class of gap: a module
can look usable (task 6), well-engineered (task 7), and functionally deep (task 8) while still
having a broken **chain** -- e.g. a real UI and a real API that don't actually reach a real DB
table, or a real workflow with no audit trail. This task checks **coverage of the end-to-end chain
only** -- it explicitly does **not** judge quality. A UI that is ugly but really wired to a real
API/DB/workflow counts as covered; a UI that is beautiful but calls a route that 404s does not.

## Important: this is a single systematic sweep, not a 3-perspective exercise
Unlike tasks 6-8, do **not** apply the repo's 3-independent-passes-then-reconcile methodology here
-- there are no differing perspectives to reconcile against each other for a coverage checklist
like this one. Instead, build the matrix once, then do a **second independent verification pass
over the same matrix** (re-check each cell yourself, especially any cell you marked "yes" quickly)
before finalizing, to catch cells you rushed or guessed on the first pass. State plainly which
cells changed between your first build and your verification pass, and why.

## What you are actually being asked to build
For every real business capability you can identify in VERIDIAN, build one row in a traceability
matrix with these exact columns:

| Business Capability | UI | API | Service | DB | Workflow | Report | Dashboard | Audit Log | Status |
|---|---|---|---|---|---|---|---|---|---|

For each column, mark **Yes** (name the real file/table/route you confirmed), **No** (you searched
and found nothing), or **Partial** (name what exists and what's missing). The **Status** column is
a one-word roll-up: **Complete** (all 8 columns Yes), **Broken Chain** (at least one Yes-then-gap in
the middle, e.g. UI and API exist but no DB write), or **Not Built** (mostly No across the row).

## Business capabilities to cover -- a real starting list, verify and extend it yourself
Do not treat this list as complete or as already-confirmed to exist -- it is a starting point built
from real module names found on 2026-07-30 in `FChecklist/compliance-tracker`'s
`src/app/(app)/erp/` and `src/app/(app)/` directories; confirm each capability actually exists
before scoring it, and add any other real, distinct business capability you find that isn't listed
here (aim for at least 15 rows total):

1. Create Project
2. Create Purchase Requisition / Purchase Order (`erp/procurement`, `src/app/api/erp/buying/
   purchase-orders`, `src/lib/services/erp-procurement-workflow-service.ts`,
   `src/lib/engines/procurement-engine.ts`)
3. Approve Purchase Order
4. Record Goods Receipt (`erp/goods-receipt`)
5. Post Vendor/Purchase Invoice (`src/app/api/erp/purchase-invoices`)
6. Issue Purchase Credit Note (`src/app/api/erp/purchase-credit-notes`)
7. Post Journal Entry (`erp/journal-entries`)
8. Run Payroll (`erp/payroll`)
9. Create Fixed Asset Record (`erp/fixed-assets`)
10. Reconcile Bank Statement (`erp/bank-reconciliation`)
11. Create Site Diary Entry (`site-diary`)
12. Log RFI (`rfis`)
13. Approve Change Order (`change-orders`)
14. Create Punch List Item (`punch-list`)
15. Onboard/Register Vendor (`erp/suppliers`)
16. Onboard Customer (`erp/customers`)
17. Generate a Financial or Construction Report (`reports`, `src/lib/services/
    report-catalog-service.ts`, `erp-financial-report-service.ts`, `construction-reports-service.ts`)
18. Any real HR capability you confirm exists (e.g. leave request, recruitment stage change --
    `hr`, `recruitment`, `leave-holiday`)

## How to fill in each column -- be concrete
- **UI**: a real page/component path under `src/app/(app)/...` or `src/components/...`.
- **API**: a real route path under `src/app/api/...` -- open the handler, don't assume from the
  folder name alone.
- **Service**: a real file under `src/lib/services/` (or `src/lib/engines/`) that the API route
  actually calls -- trace the import, don't guess from naming similarity.
- **DB**: a real table defined in a `drizzle/*.sql` migration, confirmed to actually be written to
  by the service above (not just a plausibly-named table that happens to exist).
- **Workflow**: a real multi-step state transition (e.g. draft -> submitted -> approved), backed by
  a real status field/state machine in code -- not just a single CRUD operation.
- **Report**: a real report that includes this capability's data, confirmed by opening the report
  service/route, not assumed from the module name.
- **Dashboard**: confirmed presence of this capability's data on a real dashboard
  (`dashboard`, `construction-dashboard`, `kpi-hub`) -- open the dashboard code/page, don't assume.
- **Audit Log**: a real, persisted audit trail entry for this action. Start from
  `src/lib/activity-log-service.ts` (compliance-tracker) -- its own header comment (as of
  2026-07-30) states it is scoped to only one activity type (`ai_team_dispatch`) so far, which
  means most business actions likely have **no** entry there; confirm whether any *other* real
  audit-log mechanism exists for the capability you're checking (e.g. a dedicated `audit_log` table
  written directly by the service, separate from `activity-log-service.ts`) before marking this
  column Yes.

## What NOT to do
- Do not mark a column Yes because a plausibly-named file exists -- open it and confirm it's
  actually wired to the neighboring links in the chain (e.g. the API route actually calls that
  service; that service actually writes to that table).
- Do not judge quality, usability, or correctness of what you find here -- a row can be "Complete"
  even if the UI is clunky or the report is ugly, as long as the chain genuinely connects end to
  end. Save quality judgments for tasks 6-8.
- Do not skip the Audit Log column or wave it through as Yes by default -- this repo's own
  same-day investigation (see the pointer above) found this is a real, likely-widespread gap; treat
  every row's Audit Log cell as requiring its own real check.
- Do not stop at the 15+ starting capabilities if you find other real, distinct capabilities worth
  a row -- add them.

## Output
Produce the full matrix (at least 15 rows), a one-paragraph summary of the most common broken-chain
pattern you found (e.g. "Report and Dashboard columns are the most frequent gap; Audit Log is
gapped almost everywhere"), and a note on what changed between your first pass and your
verification pass. Write your findings to `findings/repository-traceability/` (see that folder's
`README.md` for the exact expected shape). End with
`STATUS: repository-traceability-audit COMPLETE` or
`STATUS: repository-traceability-audit BLOCKED -- <one-sentence reason>`.
