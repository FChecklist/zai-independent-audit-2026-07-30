---
task_id: multitenant-enduser-perspective-audit
priority: 3
findings_folder: findings/multitenant-enduser-perspective/
status: pending
---

# Task 3: Multi-tenant end-user perspective audit

## The question
Forget the Owner's and the deterministic-software lens for this task. Pretend you are simply a
**real end user** of this product, in a system that claims to be multi-tenant SaaS. Does it
actually work correctly: real task flow, real UI, and -- most importantly -- real data isolation
between tenants?

This is a genuinely different lens from tasks 1 and 2: this session's own prior work has focused
more on the Owner/business side and on internal code-logic review, and has done comparatively
little of this "act as an end user against the live system" style of testing. Your independent
value here is highest.

## Where to actually look and test (real, checked 2026-07-30)

- **Live production site:** `https://projexa-ai.com` -- confirmed reachable (HTTP 200) by an
  independent check on 2026-07-30. This is a real, running deployment, not a mockup. Use your own
  browsing capability to actually load it and interact with it.
- **Demo tenant accounts exist for exactly this purpose** (multiple separate demo organizations set
  up specifically to test tenant isolation). Their credentials are **deliberately not written
  anywhere in this repo** (this repo is public). Ask the Owner, in your chat session, to give you
  the demo login credentials directly before starting this task. If you don't have them, say so in
  your findings as `BLOCKED` rather than fabricating a login attempt.
- **Source for how tenant scoping is supposed to work:** `FChecklist/compliance-tracker` (PUBLIC
  repo) -- look at `src/lib/db/schema.ts` for how records are modeled (is there an `orgId` /
  `tenantId` column on the tables that matter?), and search the `src/lib/services/*.ts` files for
  the query functions that read/write those tables -- do they actually filter by the current org
  on every read and write, or are there functions that skip that filter?

## What to actually determine (do these as real actions where you have the access, not thought experiments)

1. **Sign-up / login flow**: does the real login/signup flow on `projexa-ai.com` work as an
   anonymous visitor would experience it? Note anything broken, confusing, or inconsistent with
   what a production SaaS product should have (password reset, error messages, session handling).
2. **Cross-tenant data isolation, tested for real**: using two different demo tenant accounts (once
   the Owner gives you credentials), create or view a record as tenant A, then log in as tenant B
   and attempt to view/access that same record (by ID guessing in a URL if the UI exposes IDs, by
   checking lists/search, by any other real means available to you as a browsing user). Does
   tenant B ever see tenant A's data? Report the literal steps you took and the literal result.
3. **Source cross-check**: for whichever specific feature you tested live in step 2, find the
   actual server-side function/route in `compliance-tracker` that serves that data, and confirm
   whether it has a real org/tenant filter in its query -- quote the actual filter condition (or
   note its absence).
4. **Real task flow**: pick one core workflow a genuine end user would do (e.g. create a project,
   add a task, invite a teammate -- whatever the product actually offers) and walk it end-to-end as
   a real user would, noting anywhere it breaks, dead-ends, or behaves unexpectedly.

## What NOT to do
- Do not just read the schema and declare tenant isolation "should be fine" -- you have live access
  to the actual product; use it. A code-only claim here is a much weaker finding than a real tested
  cross-tenant access attempt with a literal before/after result.
- Do not attempt to guess or brute-force credentials, exploit unrelated vulnerabilities, or access
  any account you were not given credentials for. Testing isolation with two accounts you were
  legitimately given is in scope; anything beyond that is not.
- Do not fabricate a "COMPLETE" test result if you were never given credentials -- mark it
  `BLOCKED` with the specific missing input.

## Required steps (3-pass methodology from the repo README)
1. Run the full walkthrough three independent times if credentials allow re-testing (three separate
   attempts at the cross-tenant check specifically, since this is the highest-value real test here).
2. Produce one distilled, reconciled analysis of what you found, resolving any inconsistency
   between passes (e.g. isolation held on 2 of 3 attempts -- that is itself an important finding,
   not something to average away).
3. List concrete, named gaps (a specific URL/route/record that leaked, a specific broken UI step).
4. Give close-ended remediation, e.g. "add a server-side test asserting `GET /api/<route>` returns
   403/404 when the requesting org does not match the record's `orgId`", not "add better access
   control".

## Output
Write your findings to `findings/multitenant-enduser-perspective/` (see that folder's `README.md`
for the expected shape). End with `STATUS: multitenant-enduser-perspective-audit COMPLETE` or
`STATUS: multitenant-enduser-perspective-audit BLOCKED -- <reason>`.
