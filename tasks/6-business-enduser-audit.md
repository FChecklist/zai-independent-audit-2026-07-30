---
task_id: business-enduser-audit
priority: 6
findings_folder: findings/business-enduser/
status: pending
---

# Task 6: Business & end-user audit

## Why this task exists
This is the first of four independent, parallel audits (tasks 6-9) that together form a second,
more rigorous consulting-style pass over VERIDIAN, mirroring how large consulting firms structure
a multi-workstream review -- each workstream blind to the others, reconciled only afterward (that
reconciliation is task 10, and it does not run until all four of tasks 6-9 are done). This task is
the **business and usability** workstream. The Owner gave explicit authority to edit, tighten, and
rework this audit's scope to get to a genuinely complete result -- restating the Owner's framework
back is not an acceptable answer to any question below.

## Who you are for this task
You are evaluating VERIDIAN (via its real, public application code and its live customer-facing
product PROJEXA) **purely as a business stakeholder and end user would**, adopting each of these
13 real-world perspectives in turn: **CEO, COO, Business Owner, Construction Company (the buyer
organization), Interior Designer, Project Manager, Site Engineer, Purchase (procurement staff),
HR, Finance, Vendor, Customer, End User** (any daily operator logging in to get work done).

**Explicitly ignore implementation details for this task** -- code quality, architecture, database
design, and API correctness are out of scope here (that is task 7). Judge only from a business
value / usability standpoint: could a real person in each of these roles actually use this to run
part of a real business, today, without a developer standing next to them?

## Where to actually look (real, checked 2026-07-30 -- verify independently, do not accept as confirmed)

- **`FChecklist/projexa`** (public repo as of 2026-07-30, made public for this audit -- re-check
  its current visibility yourself with `gh repo view FChecklist/projexa --json isPrivate,visibility`
  or by simply trying to open it; do not assume the state described here still holds) -- the
  customer-facing construction-ERP frontend. Real files worth opening first: `PROJEXA_GAP_ANALYSIS.md`,
  `PROJEXA_TASK_GOVERNANCE.md`, `PROGRESS.md`, `PHASE1_SEED_REPORT.md`,
  `PHASE2_BATCH_B_FINDINGS.md`, `PHASE2_BATCH_C_FINDINGS.md` at the repo root -- these are the
  product team's own claims about what's built; treat every claim in them as unverified until you
  cross-check it against real `src/` code or the live site.
- **Live production check (do this yourself, right now):** `https://projexa-ai.com`. Load it in
  your own browsing capability. If you have real demo-tenant login credentials (the Owner will
  paste them into your chat session if you ask), actually log in and use the product as each
  persona would -- do not judge only the marketing/landing pages. Note exactly what a real user can
  and cannot do.
- **`FChecklist/compliance-tracker`** (public repo) -- the underlying VERIDIAN application most of
  PROJEXA is built on via API. Its authenticated app surface lives under `src/app/(app)/` and
  contains (real, checked 2026-07-30) over 100 route folders spanning construction/PM workflows
  (`site-diary`, `rfis`, `submittals`, `punch-list`, `change-orders`, `floor-plans`, `scope`,
  `doa`), ERP/finance (`src/app/(app)/erp/` with `procurement`, `invoicing`, `payroll`,
  `fixed-assets`, `bank-reconciliation`, `budgets`, `cash-management`, `journal-entries`,
  `goods-receipt`, `suppliers`, `customers`, `returns`, `credit-notes`, `inventory`), people
  (`hr`, `hr-compliance`, `recruitment`, `leave-holiday`, `training`, `labour`), governance
  (`approvals`, `board`, `board-evaluation`, `committees`, `directors`, `doa`, `audit`,
  `audit-engagements`), and cross-cutting (`dashboard`, `construction-dashboard`, `kpi-hub`,
  `reports`, `crm`, `sales-hq`, `tickets`, `voice-tickets`, `veri-ai`, `veri-todo`). Do not assume
  this list is exhaustive or still accurate -- re-list `src/app/(app)/` yourself and treat this as
  a starting map, not ground truth.
- Reports/dashboards specifically: `src/lib/services/report-catalog-service.ts`,
  `report-engine-service.ts`, `custom-report-service.ts`, `construction-reports-service.ts`,
  `erp-financial-report-service.ts` (compliance-tracker) -- open a handful of these and judge
  whether the reports they actually produce would be useful to a CFO, PM, or site engineer, not
  whether the service file exists.
- Approvals specifically: `src/app/(app)/approvals`, `src/lib/services/approval-workflow-service.ts`,
  `src/lib/services/erp-procurement-workflow-service.ts` -- trace one real approval flow (e.g. a
  purchase order) end to end as a Purchase or Finance persona would experience it.

## What "is this a usable, complete ERP" means here, concretely

Break the Owner's question set into these closed-ended sub-questions. Answer each from the
standpoint of the relevant persona(s) named in brackets, with a real, named example (a screen, a
report, a workflow you actually traced) as evidence -- not a general impression:

1. **Usability** [End User, Site Engineer, Interior Designer]: Pick 3 real workflows a first-time
   user would need on day one (e.g. logging a site diary entry, raising an RFI, creating a punch
   list item). For each, is the path to complete it discoverable and short, or would it confuse or
   frustrate a non-technical user? Name the specific friction point if any.
2. **Business-problem fit** [CEO, COO, Business Owner, Construction Company]: Pick 3 real business
   problems a construction company has (e.g. "know true job-costing in real time", "prevent
   unapproved spend", "track subcontractor compliance"). For each, does a real, traceable feature
   set solve it today, partially, or not at all?
3. **Missing capability**: Name at least 3 concrete capabilities you'd expect in a mature
   construction/business ERP that you could **not** find any real trace of after searching (not
   "might be missing" -- something you actively searched for and did not find).
4. **Workflow completeness** [Purchase, Finance, PM]: Pick one real end-to-end workflow (e.g.
   purchase requisition -> purchase order -> goods receipt -> vendor invoice -> payment) and trace
   it through the real screens/services. Does it run start to finish, or does it stop partway
   (e.g. PO creation exists but there's no real matching/payment step)?
5. **Reports & dashboards usefulness** [CEO, COO, Finance, PM]: Open `dashboard`,
   `construction-dashboard`, and `kpi-hub`, plus 2-3 real reports from the report services listed
   above. Would a real executive or PM get an actual decision-useful answer from these, or are they
   placeholders, empty states, or too narrow to be useful?
6. **Approvals correctness** [Purchase, Finance, Business Owner]: Trace one real approval chain
   (see pointers above). Does it enforce a sensible real-world authority chain (e.g. amount-based
   escalation, delegation of authority per `doa`), or is it a single generic approve/reject with no
   real business logic behind it?
7. **"Can a company actually run on this?"** [CEO, COO, Business Owner]: Considering everything
   above, give a direct verdict -- could a real construction company or interior design firm run
   its actual day-to-day operations on this today, with a small pilot team? What is the single
   biggest blocker if not?
8. **What will frustrate users** [Vendor, Customer, End User, HR]: Name the 3 most concrete,
   evidenced sources of user frustration you found (a confusing flow, a missing notification, a
   dead-end screen, a report that doesn't load real data) -- not generic "onboarding could be
   better"-style comments.

## What NOT to do
- Do not evaluate code quality, folder structure, database design, or API design here -- that is
  task 7's job. If you notice something like that, you may mention it in passing but it must not
  substitute for a business/usability answer.
- Do not accept `PROJEXA_GAP_ANALYSIS.md`, `PROGRESS.md`, or any other self-reported status doc's
  claims at face value -- treat them as claims to verify against real screens/code/the live site.
- Do not pad the report with generic ERP-maturity commentary that isn't grounded in something you
  actually opened, clicked, or traced in this specific system.
- Do not invent access you don't have -- if you cannot log into the live site or cannot open a
  repo, say so plainly rather than guessing what you would have seen.

## Required steps (3-pass methodology from the repo README)
1. Run this full 13-perspective, 8-question analysis **three independent times** -- re-derive each
   pass fresh rather than repeating your first answer with different wording. Reasonable variation
   between passes is expected (e.g. which 3 workflows you pick); that variation is useful signal,
   not noise to hide.
2. Produce **one** distilled, reconciled analysis: where the three passes agree, say so plainly;
   where they disagree (e.g. one pass judged a workflow complete, another found it broken),
   resolve it with evidence, not by picking one arbitrarily.
3. Identify concrete, named gaps -- a specific missing workflow step, a specific unusable report, a
   specific persona whose needs are unmet.
4. Give close-ended remediation measures only -- e.g. "add a payment-status field and a matching
   UI badge to the vendor-invoice screen so Finance can see unpaid invoices without opening
   `journal-entries` separately," not "improve the finance module."

## Output
Write your findings to `findings/business-enduser/` (see that folder's `README.md` for the exact
expected shape). End with `STATUS: business-enduser-audit COMPLETE` or
`STATUS: business-enduser-audit BLOCKED -- <one-sentence reason>`.
