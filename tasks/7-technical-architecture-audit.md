---
task_id: technical-architecture-audit
priority: 7
findings_folder: findings/technical-architecture/
status: pending
---

# Task 7: Technical architecture audit

## Why this task exists
This is the second of four independent, parallel audits (tasks 6-9) forming the second,
consulting-style audit pass over VERIDIAN. This task is the **engineering** workstream, run blind
to task 6's business findings (reconciliation happens only in task 10, after all of 6-9 are done).
The Owner gave explicit authority to edit, tighten, and rework this audit's scope -- restating the
framework back is not an acceptable answer to any question below.

## Who you are for this task
You are evaluating VERIDIAN purely as an engineer would, adopting each of these perspectives in
turn: **CTO, Enterprise Architect, Senior Software Architect, Database Architect, DevOps, Security,
SaaS Architect**.

**Explicitly ignore business concerns for this task** -- whether a feature is useful to a CEO or a
site engineer is out of scope here (that is task 6). Judge only engineering quality: is this
correctly built, and would you be comfortable being the engineer on call for it?

## Where to actually look (real, checked 2026-07-30 -- verify independently, do not accept as confirmed)

- **Scale reality, so you calibrate expectations correctly:** `FChecklist/compliance-tracker`
  (public) contains, as of a same-day count on 2026-07-30: 139 top-level folders under
  `src/app/api/`, 295 files under `src/lib/services/`, and 270 `*.sql` files under `drizzle/`. A
  "technical debt" or "duplication" finding on a codebase this size needs a real, named example --
  general "large codebases can have debt" commentary is not evidence.
- **Multi-tenant isolation**: `src/lib/services/tenant-isolation.test.ts` and
  `src/lib/services/projexa-records-tenant-isolation.test.ts` (compliance-tracker). Separately, a
  same-day grep found 133 files under `drizzle/*.sql` containing the literal string
  `ROW LEVEL SECURITY` -- re-run that grep yourself (`grep -rl "ROW LEVEL SECURITY" drizzle/*.sql`)
  and treat the count as a starting hint, not confirmed current fact. Pick 5 real tables with RLS
  policies and check: does the policy actually key off `org_id = current_org_id()` (or equivalent),
  and is there any real table storing tenant data that has **no** RLS policy at all (a genuine
  isolation gap, not just an inconsistency)?
- **DB normalization / schema quality**: the 270 migration files in `drizzle/*.sql` are the real,
  authoritative schema history. Cross-reference against `ai-os/DATABASE_CATALOG.json`
  (compliance-tracker) exactly as task 4 does -- pick 5 tables and confirm the catalog matches the
  real `CREATE TABLE`/`ALTER TABLE` statements. Separately, judge normalization on its own terms:
  pick 3 real tables and check for obvious anti-patterns (repeated/denormalized JSON blobs where a
  join would be correct, missing foreign keys, missing indexes on obviously-hot columns).
- **API correctness**: pick 5 real routes under `src/app/api/` (e.g. `src/app/api/erp/buying/
  purchase-orders`, `src/app/api/erp/procurement`, `src/app/api/v1/projexa/purchase-orders`,
  `src/app/api/notifications`, `src/app/api/reports/catalog/route.ts`). For each, open the actual
  route handler: does it validate input, enforce auth/tenant scoping, and return consistent error
  shapes? Note any route that skips these.
- **Code maintainability / duplication**: `src/lib/browser-execution/` (compliance-tracker) is a
  real, concrete, named example worth checking first -- task 5 of this repo independently found a
  merged feature (PR #586, files `tier-detection.ts`, `tier-orchestrator.ts`, `client-compile.ts`,
  etc.) and a separate, still-open, unmerged PR (#584) touching a different-but-overlapping file
  set (`tier-capabilities.ts`, `tier-runners.ts`, `storage-cache.ts`, etc.) for what looks like the
  same feature. Verify whether that duplication situation is still real, and search for other real
  instances of the same pattern (two services/files doing near-identical work) elsewhere in
  `src/lib/services/` (295 files is a large enough surface that near-duplicates are plausible --
  find real ones, don't assume).
- **AI integration cleanliness**: `src/lib/ai-router/`, `src/lib/ai-team/` (compliance-tracker).
  Is there one clear, consistently-used entry point for LLM calls, or multiple parallel/competing
  mechanisms? Cross-check against `ai-os/AI_ENGINEERING_POLICY.yaml` and this repo's own task 2
  findings once they exist (do not wait for them -- verify independently now, cross-reference
  later if useful).
- **Security**: search for hardcoded secrets/keys (`grep -rn "sk-\|api_key\s*=\|password\s*="` style
  patterns across `src/`, understanding this is a heuristic not a guarantee), check
  `src/lib/abac.ts` (attribute-based access control) and its test file `src/lib/abac.test.ts` for
  real enforcement vs. a thin wrapper, and check whether authentication/session handling
  (`src/app/auth/`, `src/app/mfa-challenge/`) has real, tested coverage.
- **Metadata completeness**: `ai-os/MASTER_INDEX.yaml`, `ai-os/FUNCTION_CATALOG.json`,
  `ai-os/DATABASE_CATALOG.json` (compliance-tracker) -- this repo's task 4 already covers these in
  depth from an indexing-architecture angle; for this task, judge them narrowly as **metadata
  completeness**: do real entities (tables, functions, routes) actually have owner/description/
  quality-score/lineage populated, or mostly blank/default values?
- **Scalability**: look for any real evidence of load/perf testing (`e2e/` folder, any
  `*.perf.ts`/`*bench*` files), connection pooling configuration (`drizzle.config.ts`, any DB pool
  size settings), and background-job/queue infrastructure vs. purely synchronous request handling.

## What you are actually being asked to determine

For each of the following dimensions, give a named, evidenced verdict -- not a general impression:

1. **Architecture correctness** -- is there a coherent layering (UI -> API -> service -> DB) that's
   actually followed, or do routes reach directly into the DB, bypassing the service layer, in
   places you can name?
2. **Folder structure** -- is `src/app/(app)/` (100+ route folders) organized in a way a new
   engineer could navigate, or is it a flat sprawl with no grouping/domain boundaries?
3. **Code maintainability** -- name at least one real file or service you'd flag as too large,
   too tangled, or doing too many unrelated things (with a real line count or responsibility list).
4. **DB normalization** -- your verdict from the pointer above, with named tables.
5. **Multi-tenant correctness** -- your verdict from the RLS pointer above, with named tables
   (including any real gap you found).
6. **Security** -- your verdict, with named files/patterns checked.
7. **API correctness** -- your verdict, with named routes.
8. **Performance** -- your verdict, with named evidence (or named absence of evidence).
9. **Metadata completeness** -- your verdict, with named catalog entries checked.
10. **AI integration cleanliness** -- your verdict, with named entry points.
11. **Duplication** -- your verdict on the PR #584/#586 situation (still real or resolved?) plus
    any other real duplicate you found.
12. **Technical debt** -- name the single largest concrete piece of technical debt you found (a
    real file, a real pattern), not a category.
13. **Scalability** -- your verdict, with named evidence.

## What NOT to do
- Do not evaluate whether a feature is useful to a business user -- that is task 6's job.
- Do not accept any catalog file's "live"/"complete" self-description at face value -- open the
  named entities and check.
- Do not pad the report with generic software-architecture best-practice commentary that isn't
  grounded in a real file or pattern you actually found in this codebase.
- Do not report a codebase-scale statistic (e.g. "295 service files") as itself a finding -- it is
  context; the finding is what you found when you actually opened a sample of them.

## Required steps (3-pass methodology from the repo README)
1. Run this full 13-dimension analysis **three independent times** -- re-derive each pass fresh,
   ideally sampling a different set of files/routes/tables each pass to widen real coverage, rather
   than repeating the same evidence with different wording.
2. Produce **one** distilled, reconciled analysis across all 13 dimensions, resolving any
   disagreement between passes with evidence.
3. Identify concrete, named gaps.
4. Give close-ended remediation measures only -- e.g. "add a CI check that fails the build if a new
   table migration lacks a matching `ENABLE ROW LEVEL SECURITY` statement in the same migration
   file," not "improve security posture."

## Output
Write your findings to `findings/technical-architecture/` (see that folder's `README.md` for the
exact expected shape). End with `STATUS: technical-architecture-audit COMPLETE` or
`STATUS: technical-architecture-audit BLOCKED -- <one-sentence reason>`.
