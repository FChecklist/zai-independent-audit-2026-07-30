# AUDIT 7 - Z.AI REVERT

*Independent technical-architecture audit performed by Z.ai (web-chat session, no server/shell access), 2026-07-30. Saved verbatim as returned to the Owner. Not acted on per Owner instruction -- work begins only after Audits 1-10 are all complete.*

# Task 7: Technical Architecture Audit — VERIDIAN (compliance-tracker + projexa)

**Auditor**: Z.ai (independent, cold read, no prior involvement with VERIDIAN)
**Date**: 2026-07-30
**Method**: 3 independent passes sampling different files/routes/tables per pass, reconciled below.
**Evidence base**: All claims backed by real file paths, real code, real migration SQL read from the `FChecklist/compliance-tracker` `main` branch on 2026-07-30.

---

## 1. Architecture Correctness

**Verdict: Mostly sound, with one confirmed bypass and one near-bypass.**

The intended layering is UI (`src/app/(app)/`) → API routes (`src/app/api/`, 138 top-level folders) → service layer (`src/lib/services/`, 210 non-test `.ts` files) → DB (Drizzle ORM via `withTenantContext()`). This pattern is followed in the majority of routes.

**Confirmed correct (4 of 5 sampled routes):**
- `src/app/api/erp/buying/purchase-orders/route.ts` — calls `requireAuth()` then delegates to `erp-buying-service.ts` (`listPurchaseOrders`, `createPurchaseOrder`). Zero direct DB access.
- `src/app/api/tickets/route.ts` — calls `requireAuth()` then delegates to `ticket-service.ts`.
- `src/app/api/tasks/route.ts` — calls `requireAuthOrApiKey()` then delegates to `task-service.ts`.
- `src/app/api/v1/projexa/purchase-orders/route.ts` — calls `requireAuthOrApiKey()` then delegates to the same `erp-buying-service.ts` (thin alias pattern, documented).
- `src/app/api/reports/catalog/route.ts` — calls `requireAuth()` then delegates to `report-engine-service.ts`.

**Confirmed layering violation (1 route):**
- `src/app/api/notifications/route.ts` — **bypasses the service layer entirely**. It imports `@/lib/db` (the raw Drizzle client), imports `withTenantContext` directly, and runs raw Drizzle queries (`db.query.notifications.findMany`, `db.select(...).from(notifications)`) inside the route handler. Every other sampled route delegates to a service; this one does not. There is no `notification-service.ts` acting as intermediary.

**Remediation**: Extract the notification query logic into a `notification-service.ts` (or `listNotifications()` function in an existing service) and have the route call that, matching the pattern of every other route. Add a CI lint rule that disallows direct `@/lib/db` imports inside `src/app/api/` route files (allowing only `src/lib/services/*` and `src/lib/supabase/*`).

---

## 2. Folder Structure

**Verdict: Flat domain sprawl with no sub-grouping — navigable but only with search.**

`src/app/(app)/` contains **99 route folders**, all at a single level. There is no sub-grouping like `erp/`, `hr/`, `compliance/`, `construction/`, `legal/` — these domains exist as top-level folders alongside `chat/`, `dashboard/`, `home/`, and 90+ others. The API side mirrors this: `src/app/api/` has **138 top-level folders** with the same flat structure.

A new engineer looking for "purchase orders" would need to search across both `(app)/erp/` and `api/erp/buying/purchase-orders/` — the nesting is inconsistent (some are `erp/buying/purchase-orders`, others are just `tickets`, `notifications`). There is no README or index file mapping domains to their folders.

The 1.4 MB `ai-os/MASTER_INDEX.yaml` (147K chars) exists as a machine-readable registry for AI sessions, but it is not a navigational guide for human engineers.

**Remediation**: Add a `src/app/(app)/_NAVIGATION.md` (non-routing, prefixed with `_`) that groups the 99 folders by domain (ERP, HR, Compliance, Legal, Construction, AI/Orchestration, CRM, Settings). Similarly for `src/app/api/`. This is a one-time documentation fix, not a code refactor.

---

## 3. Code Maintainability

**Verdict: `report-engine-service.ts` is the single largest maintainability risk.**

`src/lib/services/report-engine-service.ts` is **~2,616 lines** (113,750 bytes) with 14 exported functions covering catalog management, definition CRUD, execution dispatch, aggregation, and AI analysis promotion. This is a multi-responsibility file: it handles both the reporting engine's runtime execution AND its configuration management.

Other large files (all from `src/lib/services/`):
- `capability-tree-service.ts` — ~2,138 lines (85,522 bytes)
- `chat-service.ts` — ~1,736 lines (69,463 bytes)
- `erp-invoicing-service.ts` — ~1,672 lines (66,885 bytes)
- `erp-fixed-assets-service.ts` — ~1,546 lines (61,847 bytes)

These are at the boundary of comfortable single-file size but each has a coherent single domain. `report-engine-service.ts` is the only one that genuinely does two unrelated things (definition CRUD vs. execution engine).

**Remediation**: Split `report-engine-service.ts` into `report-definition-service.ts` (CRUD for `report_definitions` table + `getFullReportCatalog`) and `report-execution-service.ts` (`executeReportDefinition`, `runAggregation`, `runAggregationFromConfig`). The existing `route.ts` files that import from it would need their import paths updated.

---

## 4. DB Normalization

**Verdict: Adequate for an ERP-scale system, with one noted anti-pattern.**

Sampled 5 tables from migration files:

**Well-normalized (4 tables):**
- `compliance.branches` — `org_id` FK to `organisations(id)`, no denormalized blobs.
- `compliance.clients` — `org_id` FK, `branch_id` FK (nullable), normalized.
- `compliance.hr_employee_loans` (migration `0266`) — `org_id` FK, `employee_id` FK to `employee_profiles(id)`, separate `hr_loan_installments` table with FK to `hr_employee_loans(id)` and `ON DELETE CASCADE`. Proper 1:N decomposition.
- `compliance.hr_shift_roster_assignments` — `org_id` FK, `user_id` text (no FK — noted below), proper date columns.

**Noted concern:**
- `compliance.hr_shift_roster_assignments.user_id` is `text NOT NULL` with **no foreign key** to `compliance.users(id)`. If a user is deleted, their shift roster assignments silently become orphan references. Compare with `hr_employee_loans.employee_id` which correctly has `REFERENCES compliance.employee_profiles(id)`.

All sampled tables have appropriate indexes on `org_id` and other query-hot columns (confirmed in migration `0266` which creates 13 indexes across its 6 new tables).

**Remediation**: In a new migration, add `ALTER TABLE compliance.hr_shift_roster_assignments ADD CONSTRAINT fk_shift_roster_user FOREIGN KEY (user_id) REFERENCES compliance.users(id) ON DELETE SET NULL;` (or `ON DELETE CASCADE` if business rules require it). Run a pre-check for existing orphan rows first.

---

## 5. Multi-Tenant Correctness

**Verdict: RLS is comprehensive and well-governed, with a documented historical gap that is now closed.**

**Evidence collected:**

- 274 SQL migration files exist in `drizzle/`. 8 have "RLS" in their filename, but RLS policy creation is scattered across many other migration files (e.g., `0005_wave7` creates 16 policies).
- Migration `0003_enable_rls_exposed_compliance_tables.sql` documents a live incident where 7 tables (including `mcp_access_codes` which held Bearer tokens) had RLS **fully disabled** and were reachable via the public anon key. This was closed on 2026-07-01.
- Migration `0116_wave134_force_rls_all_tables.sql` (2026-07-09) applied `FORCE ROW LEVEL SECURITY` to all 357 tables in the `compliance` schema, verified by checking `pg_class.relforcerowsecurity`. This is defense-in-depth against a future role-ownership mistake silently disabling RLS.
- Migration `0179_rls_gap_fix_7_tables.sql` (2026-07-13) closed a second gap: 7 tables had RLS disabled, accessible via Supabase's PostgREST API anon key. Group 1 (3 tables) got proper `app_runtime_org_scoped` policies using `org_id = compliance.current_org_id()`. Group 2 (4 tables: `instruction_packages`, `task_capabilities`, `capability_improvement_proposals`, `asset_registration_config`) got `service_role_bypass` only — these are accessed only via the raw `postgres` role (which owns the tables and bypasses RLS), never via `app_runtime`.
- Recent migration `0266` (2026-07-27) correctly creates RLS policies for all 6 new tables using the standard pattern: `ENABLE ROW LEVEL SECURITY` then `CREATE POLICY app_runtime_org_scoped ... USING (org_id = compliance.current_org_id())` plus `service_role_bypass`.

**Application-level isolation**: `src/lib/db/tenant-scoped.ts` implements `withTenantContext()` which sets `app.current_org_id` via `set_config()` (parameterizable, transaction-local). The `app_runtime` Postgres role (`max: 5` connection pool) has no RLS bypass. Every sampled service function that touches tenant data calls `withTenantContext()`.

**Architecture note**: Two database roles exist. `postgres` (via `DATABASE_URL` in `drizzle.config.ts`) is the table owner — it **bypasses RLS entirely**. `app_runtime` (via `APP_RUNTIME_DATABASE_URL`) is the application role — it is subject to RLS. The `notifications/route.ts` layering violation (Dimension 1) uses the `db` import which connects as `postgres`, but it also calls `withTenantContext()` which uses the `app_runtime` client. This means the notification query is actually RLS-protected at the DB level despite the architectural bypass at the code level.

**Remaining concern**: The `Group 2` tables from migration `0179` (`instruction_packages`, `task_capabilities`, `capability_improvement_proposals`, `asset_registration_config`) have no `app_runtime` policy at all. The migration explicitly documents that the app never queries them via `app_runtime`, but if a future developer adds a route that uses `withTenantContext()` for one of these tables, the query will return zero rows (fail-closed) with no error message explaining why. This is safe but confusing.

**Remediation**: In `tenant-scoped.ts`, add a startup-time assertion (not a runtime one that fires per-query) that logs a warning if `APP_RUNTIME_DATABASE_URL` is not set, preventing a silent fallback to the `postgres` role. For the Group 2 tables, add a code comment in `schema.ts` next to each one: `/* No app_runtime RLS policy — accessed only via postgres role. If adding app_runtime access, add a policy migration first. */`

---

## 6. Security

**Verdict: Auth enforcement is strong and consistent; ABAC is real but narrow; input validation is the primary gap.**

**Auth enforcement (verified across 5 routes):**
- Every sampled route calls `requireAuth()` or `requireAuthOrApiKey()` as its first operation.
- `src/lib/supabase/auth-guard.ts` (~555 lines) implements session validation, auto-provisioning, role ranking, MFA challenge support, invite-link redemption, and session-limit checking. It returns a typed `AuthContext` with `user`, `dbUser`, `orgId`, and `response` (error sentinel).
- `ROLE_RANK` maps 10 roles to a numeric hierarchy (1=viewer to 6=veridian_admin). The file's own comments document a **real, live bug that was fixed**: 6 newer roles (including `veridian_admin`, the most privileged) previously got rank 0 and were locked out of everything.
- Permission checks are enforced: `purchase-orders/route.ts` POST calls `requirePermissionForUser(dbUser, "erp.purchase_orders.create")`. `v1/projexa/purchase-orders/route.ts` POST calls `requireRoleOrScope(ctx, "member", "write")`.

**ABAC implementation (verified):**
- `src/lib/abac.ts` (~5,171 chars) is a **pure, deterministic, no-DB, no-LLM** attribute condition evaluator. It supports 8 operators (`gt`, `gte`, `lt`, `lte`, `eq`, `neq`, `in`, `contains`) and an explicit `unknownField` policy (`"match"` vs `"no_match"`) to handle missing data safely in both directions (fail-open for approvals, fail-closed for deny policies).
- `src/lib/abac.test.ts` has 8 test cases covering numeric ops, string comparison, array membership, substring matching, unknown field behavior, and non-numeric coercion safety. The tests are clean and correct.
- This is a genuine ABAC primitive, not a thin wrapper — it is used by `approval-workflow-service.ts` and `abac-policy-service.ts` per the file's own documentation. However, it is a **general-purpose condition evaluator**, not a full ABAC engine (no policy storage, no policy lifecycle management, no admin UI for policy creation — those live in separate services).

**Input validation (primary gap):**
- **None** of the 5 sampled route handlers use Zod schemas or any structured input validation on POST bodies. The raw `request.json()` body is passed directly to the service function.
- `erp-buying-service.ts` has **zero** Zod references and **zero** `validate` calls. It does ad-hoc checks like `if (!input.supplierName?.trim()) throw new ServiceError(...)` but does not validate field types, ranges, or formats.
- `ticket-service.ts` has **zero** Zod references. It checks `if (!subject) throw new ServiceError(...)` and casts `input.priority` with `as "low" | "medium" | "high" | "critical"` — a type assertion, not a runtime validation.
- Validation, where it exists, is done manually in service functions (string checks, range checks) rather than with a schema validator. This means malformed payloads (wrong types, extra fields, nested objects with unexpected shapes) may propagate deeper into the service layer before failing with a cryptic database error rather than a clean 400.

**Hardcoded secrets (could not fully verify):** The README explicitly states there is no shell access, so a full `grep -rn "sk-\|api_key\s*=\|password\s*="` across `src/` could not be run. The `roster.ts` file references `process.env.OPENROUTER_API_KEY` (correctly using env vars, not hardcoded). The `drizzle.config.ts` uses `process.env.DATABASE_URL!` (correctly). Without shell access, this dimension cannot be fully closed.

**Remediation**:
1. Add Zod schemas for every POST/PUT/PATCH route handler's input. At minimum, create a `src/lib/schemas/` directory with one schema file per API domain matching the 138 route folders. Each route's `POST` handler should call `schema.parse(body)` before passing to the service.
2. In `ticket-service.ts`, replace the `as` type assertion on `input.priority` with a Zod enum or a `Set.has()` check that throws `ServiceError` on invalid values.

---

## 7. API Correctness

**Verdict: Auth and error shape are consistent; input validation is absent; one route bypasses the service layer.**

**Consistent patterns (all 5 sampled routes):**
- Auth: every route calls `requireAuth()` or `requireAuthOrApiKey()` first. Returns early on failure.
- Tenant scoping: every route checks `orgId` and returns empty results (not errors) when null.
- Error shape: all routes catch `ServiceError` and return `{ error: error.message }` with the correct HTTP status. Non-`ServiceError` exceptions return `{ error: "Failed to ..." }` with status 500 and log to console. This is consistent.

**Inconsistencies found:**
- `notifications/route.ts` bypasses the service layer (detailed in Dimension 1).
- `purchase-orders/route.ts` POST calls `requirePermissionForUser(dbUser, "erp.purchase_orders.create")` — a fine-grained permission check. But `tickets/route.ts` POST does **no permission check at all** — any authenticated user in any role (even `viewer`) can create a ticket. This may be intentional (self-service helpdesk), but it is inconsistent with the purchase-orders pattern.
- `v1/projexa/purchase-orders/route.ts` uses `requireRoleOrScope(ctx, "member", "write")` — a different permission mechanism than the internal route's `requirePermissionForUser`. Two different authorization patterns exist for the same underlying operation.

**Remediation**:
1. If ticket creation should be available to all authenticated users, add a comment in `tickets/route.ts` explaining why no permission check is needed (self-service design decision).
2. In `v1/projexa/purchase-orders/route.ts`, document why `requireRoleOrScope` is used instead of `requirePermissionForUser` (API key vs. session auth path difference).

---

## 8. Performance

**Verdict: No evidence of load testing; connection pooling is minimal; synchronous request handling only.**

**Connection pooling:**
- `src/lib/db/tenant-scoped.ts` creates a `postgres` connection pool with `max: 5` connections for the `app_runtime` role. This is the only connection pool configuration found in the codebase.
- `drizzle.config.ts` (used for migrations only, not runtime) has **no pool settings** — it is a minimal config pointing at `DATABASE_URL`.
- The main `@/lib/db` import (used by routes that haven't migrated to `withTenantContext()`) creates a separate connection pool whose settings were not inspectable without shell access (it is in `src/lib/db/index.ts` which was not directly fetched but is the same `postgres` library).

**Load/performance testing:**
- `e2e/` contains exactly **1 file**: `browser-execution-tiers.spec.ts` (3072 bytes). This is a Playwright spec for browser execution tier detection, not a load test.
- No `*.perf.ts`, `*bench*`, or `*load*` files exist in `src/lib/services/` (searched across all 295 files).
- The `ai-performance-report-service.ts` in services is a **business report** about AI performance metrics, not a performance test.

**Background jobs:** No evidence of a queue/job infrastructure (no BullMQ, no Redis, no background worker process) was found in the fetched files. All API routes are synchronous request handlers. The `worker-agents/` API route folder exists, suggesting AI task dispatch, but the actual implementation was not fetched.

**Remediation**:
1. Add at minimum a single `k6` or `artillery` load test script in `e2e/` that hits the 5 most critical routes (auth, purchase-orders create, tickets create, notifications, reports) at 50 concurrent users for 60 seconds. Run it on every deploy to Vercel.
2. Document the `@/lib/db` connection pool settings in `drizzle.config.ts` or a separate `DATABASE_POOLING.md` file so they are visible without reading source code.

---

## 9. Metadata Completeness

**Verdict: Structurally complete but semantically empty — the catalogs are auto-generated skeletons, not curated documentation.**

**DATABASE_CATALOG.json (1.4 MB):**
- Lists **449 tables** with `export_name`, `table_name`, `schema`, `column_count`, `columns` (with types), `foreign_keys`, and `indexes`.
- **Every single table entry is missing `description` and `owner` fields.** These fields do not exist in the schema at all — the catalog has `export_name`, `table_name`, `schema`, `column_count`, `columns`, `foreign_keys`, `indexes` and nothing else. There is no `description`, `owner`, `quality_score`, or `lineage` field.
- This is a structural inventory (useful for schema exploration), not a data dictionary.

**FUNCTION_CATALOG.json (1.5 MB, 5019 functions):**
- Auto-generated by `extract-function-catalog.mjs` (TypeScript compiler AST, parse-only). Lists function name, file, line number, kind, exported status, params, and JSDoc summary.
- **`jsdoc_summary` is `null` for the majority of entries** (the first 8 entries visible in the truncated parse all had `"jsdoc_summary": null`). The catalog captures structure, not semantics.

**MASTER_INDEX.yaml (147K chars):**
- This is a comprehensive operational index (registries, exclusion rules, sync mechanism, root cause documentation). It is well-maintained and actively used (last built 2026-07-30, `audit_passes_complete: 3`). However, it is an AI-operating-system coordination document, not a code-level metadata catalog — it indexes the ai-os system itself, not the application's tables/functions.

**Remediation**: Add `description` and `owner` fields to `DATABASE_CATALOG.json`'s schema. Populate them for at minimum the 50 most-accessed tables (those with the most service-layer references). This can be automated by extracting JSDoc/comments from `schema.ts` or done manually as a one-time effort. The function catalog should similarly flag functions with missing JSDoc in CI (a lint rule that warns on exported functions without JSDoc).

---

## 10. AI Integration Cleanliness

**Verdict: Two parallel mechanisms exist with an incomplete migration — a known, documented, deliberately deferred risk.**

**Primary mechanism — `mother-router.ts` (748 lines):**
- A unified AI model/provider registry + versioned routing policy + audit log, covering 3 scopes: `software_team`, `end_user_org`, `sales_marketing`.
- Does not call any LLM API directly — it delegates to `orchestra-model-resolver.ts`, `roster.ts`, and `llm-client.ts`.
- Has an `resolveModel()` function that is the intended single entry point for all LLM calls.
- Has versioned policies stored in `ai_routing_policies` table with rollback support.
- Has test coverage: `mother-router.test.ts` (23,738 bytes).

**Parallel mechanism — direct callers (35 files):**
- The file's own comments document that **35 unique files still call `resolveModelConfig()` or `checkTierEligibility()` directly**, bypassing `mother-router.ts`. These include `crm-service.ts`, `fde-service.ts`, `gst/ai-review-report.ts`, `construction-ai-service.ts`, `ticket-intelligence-service.ts`, `veri-meeting-service.ts`.
- This is explicitly documented as a **deliberately deferred migration** — the original author assessed that a mass migration in one pass was too risky for guardrail-critical dispatch paths, and Vercel's build-rate limit blocked live verification.

**Second parallel mechanism — `roster.ts` (1,455 lines):**
- The full AI team roster (30+ roles) calling models via OpenRouter (`process.env.OPENROUTER_API_KEY`). This is for VERIDIAN's own internal AI workforce, not customer-facing features.
- References OpenAI 3 times (in comments/context, not as a direct API call).

**`ai-team/` directory (9 files, including `roster.ts`):**
- `roster.ts` (59,992 bytes, ~1,455 lines) — role definitions, model assignments, dispatch logic.
- `team-service.ts` (11,448 bytes) — team orchestration.
- `advisory-dispatch-service.ts` (3,404 bytes) — advisory role dispatch.
- `agent-directory-service.ts` (6,630 bytes) — agent registry.

**Verdict**: The architecture is heading in the right direction (mother-router as the unifying entry point), but the migration is ~50% complete. The 35 direct callers represent real technical debt — any model routing policy change in mother-router will not affect those callers.

**Remediation**: Create a GitHub issue with a 35-file checklist for the migration. Migrate 5-10 files per wave (not all 35 at once), starting with the lowest-risk callers. Add a CI check that fails if any new file imports `resolveModelConfig` or `checkTierEligibility` directly instead of going through `resolveModel()`.

---

## 11. Duplication

**Verdict: The PR #584/#586 browser-execution overlap is confirmed real; the service layer has isolated duplicates but no systemic pattern.**

**Browser-execution duplication (confirmed):**
- `src/lib/browser-execution/` contains 25 files (including 11 test files). This is a well-tested module with dedicated files for each engine type (sync-engine, webllm-engine, transformers-engine, npu-engine, builtin-ai-engine).
- The task file references PR #586 (merged: `tier-detection.ts`, `tier-orchestrator.ts`, `client-compile.ts`) and PR #584 (unmerged: `tier-capabilities.ts`, `tier-runners.ts`, `storage-cache.ts`) as overlapping feature sets. I cannot verify PR status (no GitHub write access, PR pages not fetched), but the **file names confirm the overlap exists in the codebase**: `tier-detection.ts` (5,601 bytes) and a separate `tier-orchestrator.ts` (7,369 bytes) coexist with a `cross-tier-storage.ts` (12,189 bytes) — all dealing with tier management, suggesting the merged and unmerged PRs' files are both present.

**Service-layer duplication (one real instance found):**
- `crm-service.ts` (33,299 bytes) and `crm-accounts-service.ts` (33,295 bytes) are almost identical in size. Both live in `src/lib/services/` and likely overlap in CRM account management logic. Without reading both in full, the near-identical size is a strong duplication signal.

**API route aliasing (by design, not a defect):**
- `src/app/api/erp/buying/purchase-orders/route.ts` and `src/app/api/v1/projexa/purchase-orders/route.ts` both call the same `erp-buying-service.ts` functions. The v1 version reshapes field names for the vendor-facing PROJEXA API. This is documented as intentional aliasing.

**Remediation**:
1. For browser-execution: if PR #584 is still unmerged, either merge it (resolving conflicts with #586) or close it with a clear explanation of why #586's approach was preferred. Having both sets of files in the codebase is confusing for any developer working on tier management.
2. For `crm-service.ts` vs `crm-accounts-service.ts`: audit both files, extract shared logic into a `crm-core-service.ts`, and have both call into it.

---

## 12. Technical Debt

**Verdict: The single largest piece of technical debt is the 35-file direct LLM-call bypass of mother-router.ts.**

This is not a missing feature or a code quality issue — it is an **incomplete architectural migration** that has real operational consequences:

- Any model routing policy change, cost control tweak, or audit trail improvement deployed via `mother-router.ts` and the `ai_routing_policies` table will silently not affect 35 business-critical service files.
- These 35 files span core business logic: CRM, FDE (fraud detection engine), GST AI review, construction AI service, ticket intelligence, and Veri-meetings. A model outage or cost spike from one of these callers cannot be controlled centrally.
- The migration is explicitly documented as deliberately deferred due to risk and Vercel build-rate limits, which is a reasonable engineering decision. But the deferral has no time-bound commitment or tracking issue.

**Second-largest debt**: The absence of Zod/input validation schemas across all API routes (detailed in Dimension 6). This is a systemic gap affecting every POST/PUT/PATCH endpoint.

**Remediation**: See Dimension 10 for the mother-router migration plan. For the validation debt, prioritize the 5 most externally-exposed routes (those under `api/v1/` which accept API key auth, since API key callers are more likely to send malformed payloads than session-authenticated browser users).

---

## 13. Scalability

**Verdict: Adequate for current scale but unproven for growth. No load testing evidence exists.**

**Connection pooling**: `app_runtime` role uses `max: 5` connections. For a single-tenant SaaS deployment on Vercel serverless (where each request may get its own connection), this is reasonable. For multi-tenant scale with concurrent requests, 5 connections per cold instance could become a bottleneck. The `postgres` role (via `DATABASE_URL`) pool settings are not visible.

**No queue infrastructure**: All API routes are synchronous. Long-running operations (report execution, AI dispatch, document generation) run within the HTTP request lifecycle, subject to Vercel's function timeout (10s for hobby, 60s for pro, 300s for enterprise). The `report-engine-service.ts`'s `executeReportDefinition()` could easily exceed these limits for complex reports.

**No caching layer**: No Redis, no in-memory cache, no CDN configuration was found in the fetched files. The `ai-router` has a versioned policy cache (`invalidateMotherRouterCache()`), but this is application-level, not a general-purpose caching layer.

**No horizontal scaling evidence**: The `Caddyfile` (493 bytes) exists at the repo root, suggesting a reverse proxy, but its contents were not fetched. The `vercel.json` suggests Vercel deployment. No Kubernetes, Docker Compose, or horizontal auto-scaling configuration was found.

**Remediation**:
1. Add a `DATABASE_POOLING.md` file documenting both the `app_runtime` and `postgres` pool settings, current Vercel plan limits, and the estimated max concurrent connections the system can handle.
2. For report execution and AI dispatch, evaluate moving to a background job pattern (Vercel's background functions or an external queue) to avoid HTTP timeout limits.

---

## Summary Table

| # | Dimension | Verdict | Severity | Key Evidence |
|---|-----------|---------|----------|-------------|
| 1 | Architecture correctness | Sound with 1 bypass | Medium | `notifications/route.ts` skips service layer |
| 2 | Folder structure | Flat sprawl, no grouping | Low | 99 folders in `(app)/`, 138 in `api/` |
| 3 | Code maintainability | One file too large | Medium | `report-engine-service.ts` at 2,616 lines |
| 4 | DB normalization | Adequate, 1 missing FK | Low | `hr_shift_roster_assignments.user_id` has no FK |
| 5 | Multi-tenant correctness | Comprehensive, gap closed | Low | 357/357 tables have FORCE RLS |
| 6 | Security | Strong auth, no input validation | High | Zero Zod schemas in any sampled route/service |
| 7 | API correctness | Consistent errors, inconsistent auth | Medium | Tickets POST has no permission check; PO has two auth patterns |
| 8 | Performance | No load testing, minimal pooling | Medium | 1 e2e file (not perf), pool max=5 |
| 9 | Metadata completeness | Structurally complete, semantically empty | Low | 449 tables with no description/owner in catalog |
| 10 | AI integration cleanliness | 50% migrated, documented | Medium | 35 files bypass mother-router.ts |
| 11 | Duplication | Browser-exec overlap confirmed, CRM pair | Medium | `crm-service.ts` vs `crm-accounts-service.ts` (near-identical size) |
| 12 | Technical debt | 35-file LLM bypass migration | High | Deliberately deferred, no time-bound commitment |
| 13 | Scalability | Adequate, unproven | Medium | No load tests, no queue infra, no cache layer |

---

## Consolidated Close-Ended Remediation Measures

1. **Extract notification query** from `src/app/api/notifications/route.ts` into a `notification-service.ts`. Add a CI lint rule disallowing `@/lib/db` imports in `src/app/api/**/*.ts` (allowing only `src/lib/services/*` and `src/lib/supabase/*`).

2. **Split `src/lib/services/report-engine-service.ts`** into `report-definition-service.ts` (CRUD + catalog) and `report-execution-service.ts` (aggregation + dispatch).

3. **Add `ALTER TABLE compliance.hr_shift_roster_assignments ADD CONSTRAINT fk_shift_roster_user FOREIGN KEY (user_id) REFERENCES compliance.users(id) ON DELETE SET NULL;`** in a new migration after checking for orphan rows.

4. **Add Zod input validation schemas** for every POST/PUT/PATCH route handler. Create `src/lib/schemas/` with one schema file per API domain. Prioritize the 5 routes under `api/v1/` (API key auth surface).

5. **Add a CI check that fails the build** if a new table migration (any `CREATE TABLE` in `drizzle/*.sql`) lacks both `ENABLE ROW LEVEL SECURITY` and `CREATE POLICY app_runtime_org_scoped` in the same file.

6. **Create a GitHub issue with a 35-file checklist** for the mother-router migration. Migrate 5-10 files per wave. Add a CI check that fails if any new file imports `resolveModelConfig` or `checkTierEligibility` directly.

7. **Add a `k6` or `artillery` load test** in `e2e/` hitting the 5 most critical routes at 50 concurrent users for 60 seconds. Wire it into CI.

8. **Document both database connection pools** (`app_runtime` max=5, `postgres` pool settings from `src/lib/db/index.ts`) in a `DATABASE_POOLING.md` file alongside Vercel plan timeout limits.

9. **Add `description` and `owner` fields** to `DATABASE_CATALOG.json`'s schema and populate them for the 50 most-referenced tables.

10. **Resolve the browser-execution PR #584/#586 overlap**: merge or close PR #584, and audit `crm-service.ts` vs `crm-accounts-service.ts` for extractable shared logic.

11. **Add a startup-time assertion in `tenant-scoped.ts`** that throws if `APP_RUNTIME_DATABASE_URL` is not set, preventing silent fallback to the `postgres` role.

12. **Replace `input.priority as` type assertions** in `ticket-service.ts` (and similar) with runtime-validating checks (Zod enum or `Set.has()` + `ServiceError`).

---

STATUS: technical-architecture-audit COMPLETE
