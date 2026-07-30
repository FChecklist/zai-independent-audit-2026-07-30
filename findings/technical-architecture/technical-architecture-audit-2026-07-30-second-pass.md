# AUDIT 8 - Z.AI REVERT

*Independent audit performed by Z.ai (web-chat session, no server/shell access), 2026-07-30. Saved verbatim as returned to the Owner. Not acted on per Owner instruction -- work begins only after Audits 1-10 are all complete.*

**Note on naming: this document's own header reads "Task 7: Technical Architecture Audit" -- the identical task/title as the write-up already saved as Audit 7 ([technical-architecture-audit-2026-07-30.md](../findings/technical-architecture/technical-architecture-audit-2026-07-30.md)). This is NOT a copy-paste duplicate: it is a second, independently-run Z.ai session covering the same 13 dimensions, sampling different files and reaching some different verdicts (e.g. this pass calls out `max: 1` DB connection pool and an 11,384-line monolithic schema.ts that Audit 7's pass did not surface, while Audit 7 surfaced the `notifications/route.ts` service-layer bypass in more detail). Saved as a distinct file, Audit 7's file left untouched, so neither independent pass overwrites the other. Flagging this honestly rather than silently reconciling or picking one -- that reconciliation is out of scope until all 10 audits are done, per standing instruction.**

**Six supporting evidence JSON files were also supplied alongside this write-up** (raw GitHub file-fetch tool outputs backing specific claims in the audit): `p2_lib_dir.json`, `p2_llm_gate.json`, `p2_schema.json`, `p3_rls_construction.json`, `p3_rls_hr.json`, `p3_func_cat.json`. These are archived alongside this file in the audit repo under `findings/technical-architecture/audit-8-evidence/` unmodified.

---

# Task 7: Technical Architecture Audit — VERIDIAN

**Auditor:** Z.ai (independent, third-party)
**Date:** 2026-07-30
**Repos examined:** `FChecklist/compliance-tracker` (main application codebase, public)
**Methodology:** Three independent passes, each sampling different files/routes/tables, reconciled below. Every finding traces to a real file, line, or pattern actually fetched and read from the repo.

---

## 1. Architecture Correctness — VERDICT: PARTIAL

**Evidence (files read across 3 passes):**

| Route File | Service Layer? | Auth? | Tenant? | Input Validation? | RBAC? |
|---|---|---|---|---|---|
| `src/app/api/erp/buying/purchase-orders/route.ts` | ✅ `erp-buying-service` | ✅ `requireAuth()` | ✅ `orgId` | ❌ raw `body` | ✅ `requirePermissionForUser` |
| `src/app/api/erp/accounts/route.ts` | ✅ `erp-accounting-service` | ✅ `requireAuth()` | ✅ `orgId` | ❌ raw `body` | ✅ `requirePermissionForUser` |
| `src/app/api/hr/attendance/route.ts` | ✅ `hr-attendance-service` | ✅ `requireAuth()` | ✅ `orgId` | ❌ raw `body` | ✅ `requirePermissionForUser` |
| `src/app/api/tasks/route.ts` | ✅ `task-service` | ✅ `requireAuthOrApiKey()` | ✅ `orgId` | ❌ raw `body` | ❌ none |
| `src/app/api/reports/catalog/route.ts` | ✅ `report-engine-service` | ✅ `requireAuth()` | ✅ `orgId` | N/A (GET) | ❌ none |
| **`src/app/api/notifications/route.ts`** | **❌ Direct DB** | ✅ `requireAuth()` | ✅ `withTenantContext` | N/A (GET) | ❌ none |

The `notifications/route.ts` handler bypasses the service layer entirely, performing direct Drizzle ORM queries (`db.query.notifications.findMany(...)`) inline in the route handler. No service file intermediates.

**Auth** is consistently enforced — every route uses `requireAuth()` or `requireAuthOrApiKey()`. The auth-guard module (`src/lib/supabase/auth-guard.ts`, 471 lines) provides centralized role-based access with a 10-level `ROLE_RANK` hierarchy.

**Input validation is absent at the route level across all checked routes.** POST handlers pass raw `request.json()` directly to service functions. No Zod schemas, no manual type checks, no validation middleware at the API boundary.

**ServiceError handling is consistent.** Every route catches `ServiceError` and maps to the correct HTTP status. This is well-implemented.

---

## 2. Folder Structure — VERDICT: PARTIAL

**Evidence:** GitHub API listing of `src/app/api/` (138 top-level folders), `src/lib/services/` (295 files), `src/app/` (36 entries).

The `erp/` subtree is genuinely well-organized: 38 sub-folders (accounts, buying, inventory, payroll, selling, etc.) with clear domain boundaries. Similarly, `crm/`, `hr/`, `fm/`, `construction/` exist as groupings.

However, **~100 top-level API folders are a flat sprawl**: `access-review`, `ai`, `approval-workflows`, `assets`, `audit-engagements`, `audit-findings`, `audit-points`, `audit`, `automation-rules`, `challans`, `charges`, `clients`, `clm`, `committees`, `compliance`, `contact`, `documents`, `email-intelligence`, `frameworks`, `glossary`, `gst-reconciliation`, `incidents`, `ingest`, `invitations`, `knowledge-base`, `kpi-hub`, `legal-matters`, `notifications`, `policies`, `risks`, `tickets`, `training`, `webhooks`, `whistleblower` — with no sub-grouping. Redundancies exist: `audit/` AND `audit-engagements/` AND `audit-findings/` AND `audit-points/` are separate top-level folders. Legacy cruft (`stage0/`, `v1/`, `rpt/`, `r/`, `fde/`) suggests unmigrated or experimental routes.

**All 295 service files live in a single flat directory** (`src/lib/services/`) with no sub-folders. Files are prefix-named (`erp-buying-service.ts`, `crm-service.ts`, `hr-attendance-service.ts`) but not grouped into domain directories.

---

## 3. Code Maintainability — VERDICT: PARTIAL

**Evidence:** Directory listing byte counts from `src/lib/services/` and `src/lib/db/`.

| File | Size (bytes) | Estimated Lines |
|---|---|---|
| `report-engine-service.ts` | 113,750 | ~2,800+ |
| `capability-tree-service.ts` | 85,522 | ~2,100+ |
| `chat-service.ts` | 69,463 | ~1,700+ |
| `erp-invoicing-service.ts` | 66,885 | ~1,650+ |
| `erp-fixed-assets-service.ts` | 61,847 | ~1,500+ |

21 service files exceed 25KB (~600+ lines). `report-engine-service.ts` at ~2,800 lines is a clear God Object candidate. The `src/lib/db/schema.ts` file is **11,384 lines** containing all 449 tables and 124 enums in a single monolithic file. There is no splitting by domain.

Positive example: `src/lib/browser-execution/tier-detection.ts` (124 lines) is clean, well-documented, single-responsibility.

---

## 4. DB Normalization — VERDICT: FAIL

**Evidence:** `src/lib/db/schema.ts` (fetched, 11,384 lines), `ai-os/DATABASE_CATALOG.json`, multiple `drizzle/*.sql` migration files.

**Foreign key constraints: 20 `.references()` calls across 449 tables.** ~95% of apparent foreign key relationships (columns named `org_id`, `user_id`, `created_by_id`, etc.) have no database-level FK constraint. The initial migration (`0000_clammy_may_parker.sql`) confirms this — `compliance_items` has `org_id text NOT NULL` and `department_id text NOT NULL` and `assigned_to_id text` but zero FK definitions.

**Zero explicit `.index()` calls in the entire schema.ts.** No indexes on frequently-queried columns like `org_id`, `user_id`, `created_at`, or status fields.

**82 of 449 tables (18%) use JSONB columns** where relational structures would be more appropriate. The worst offender is `platform.dynamic_chains` with 14 JSONB columns out of 30 total. `compliance.veri_meetings` stores `attendees`, `agenda`, `minutes` as JSONB — clear candidates for proper relational tables.

**143 of 449 tables (32%) lack an `org_id` column.** Some are legitimate (audit tables, platform config), but `ai_assistants`, `client_entities`, and `ai_model_registry` missing tenant scoping is worth investigation.

---

## 5. Multi-tenant Correctness — VERDICT: PARTIAL

**Evidence:** `tenant-isolation.test.ts` (fetched), `projexa-records-tenant-isolation.test.ts` (fetched), 275 `drizzle/*.sql` files, `src/lib/db/tenant-scoped.ts`.

**Application-level isolation works.** The tenant isolation tests exercise real service functions and verify that `withTenantContext()` is always called with the correct `orgId`. The tests are well-designed — they mock only `withTenantContext` and exercise real service code.

**Database-level RLS exists but has known gaps.** Evidence:
- `0003_enable_rls_exposed_compliance_tables.sql`: 7 tables had RLS fully disabled (including `mcp_access_codes` holding live Bearer tokens). Fixed with blanket `service_role_bypass USING (true)` policies — org-scoped RLS was deferred.
- `0116_wave134_force_rls_all_tables.sql`: Forced RLS on 357 existing compliance-schema tables, but this was a one-time DO-block loop. Every table created by a later migration required a separate FORCE operation.
- `0179_rls_gap_fix_7_tables.sql`: Fixed 7 tables where RLS was re-disabled by later migrations.
- `0215_wave_a_force_rls_construction_interior_tables.sql`: Fixed 11 post-migration tables that had RLS enabled but not forced.
- `0223_force_rls_hr_attendance_holidays.sql`: Fixed 2 more tables created after 0215, noting explicitly that *"the broader list of compliance-schema tables that are also RLS-enabled-not-forced (activity_log, monitor_task_state, report_definitions, org_join_codes, workspace_memory_capsule_events, and more) — a real, separately-deferred gap."*
- `0227_abac_policy_layer.sql`: The most recent table (`abac_policies`) correctly uses `ENABLE ROW LEVEL SECURITY` + `FORCE ROW LEVEL SECURITY` + a real `org_id = compliance.current_org_id()` policy — showing the pattern is now established for new tables.

**The gap pattern is clear:** RLS is patched reactively after audits find missing policies, not proactively enforced at migration time. The `0223` migration explicitly documents remaining tables that are "RLS-enabled-not-forced" as a deferred gap.

---

## 6. Security — VERDICT: PARTIAL

**Evidence:** `src/lib/abac.ts` (fetched, full file), `src/lib/abac.test.ts` (fetched, full file), `src/lib/supabase/auth-guard.ts` (fetched, 471 lines), `.gitignore` (fetched), `src/lib/db/connection-string.ts` (fetched).

**ABAC is real and well-designed.** `abac.ts` is a pure, deterministic multi-condition evaluator supporting 8 operators (`gt`, `gte`, `lt`, `lte`, `eq`, `neq`, `in`, `contains`). It has an explicit `unknownField` safety policy — callers must choose whether missing data fails open toward more restriction or less restriction. The test file has 6 test cases covering numeric, string, array, substring, unknown-field, and error-safety scenarios. This is genuinely well-built.

**Auth guard is substantive.** 471 lines with 10-role hierarchy (`ROLE_RANK`), auto-provisioning, license/seat checks, and session limits. The file documents a real bug that was found and fixed: 6 newer roles (including `veridian_admin`) were functionally locked out of everything because they weren't in the original 4-role `ROLE_RANK` map.

**Auth directory exists** with a `callback/` handler for OAuth flows. MFA challenge page exists at `src/app/mfa-challenge/`.

**No `.env.example` exists** (404). The `.gitignore` correctly blocks `.env*` patterns. The `connection-string.ts` file uses `process.env.DATABASE_URL` or falls back to `NEXT_PUBLIC_SUPABASE_URL + SUPABASE_DB_PASSWORD` — a documented fix from a prior audit that found one copy had the wrong Supabase region.

**No `src/middleware.ts` exists** (404). There is no Next.js middleware file for rate limiting, security headers, or request filtering at the edge. All security enforcement happens at the route-handler level.

---

## 7. API Correctness — VERDICT: PARTIAL

**Evidence:** 7 route handlers fetched across 3 passes (purchase-orders, accounts, attendance, tasks, reports/catalog, notifications, plus attempted inventory/crm/construction routes — 3 returned 404 indicating these module paths differ from expectations).

**Consistent patterns (good):**
- All routes use `requireAuth()` or `requireAuthOrApiKey()`.
- All routes return empty data arrays (not errors) when `orgId` is missing.
- All routes catch `ServiceError` with correct HTTP status mapping.
- All routes use consistent error response shapes: `{ error: "..." }`.

**Consistent gaps (bad):**
- Zero input validation at the route level. No Zod, no Yup, no manual checks.
- RBAC is not universal — tasks and notifications routes have no permission check.
- The notifications route bypasses the service layer entirely.

**Route path inconsistency:** `src/app/api/erp/procurement/route.ts`, `src/app/api/crm/route.ts`, `src/app/api/inventory/route.ts`, and `src/app/api/construction/route.ts` all returned 404. The actual paths differ from what task 7 suggested — this means the task's own file-path pointers are already stale.

---

## 8. Performance — VERDICT: FAIL

**Evidence:** `drizzle.config.ts` (fetched), `src/lib/db/index.ts` (fetched), `package.json` (fetched), `e2e/` directory listing (fetched — contains only 1 file: `browser-execution-tiers.spec.ts`), `src/lib/` directory listing.

**DB connection pool: `max: 1`.** The `postgres.js` driver in `src/lib/db/index.ts` is configured with `max: 1` — a single connection per process. On Vercel serverless, each cold start gets exactly 1 DB connection. This is confirmed by the actual code:

```typescript
client = postgres(getConnectionString(), {
  prepare: false,
  ssl: { rejectUnauthorized: false },
  max: 1,
})
```

**No performance testing exists.** The `e2e/` directory contains a single file (`browser-execution-tiers.spec.ts`, 3KB) — no `*.perf.ts`, no `*bench*`, no load testing.

**No background job/queue infrastructure.** Package.json contains zero entries for `redis`, `ioredis`, `@upstash/redis`, `bull`, `bullmq`, or any async queue. The only caching is in-process memory (`llm-response-cache.ts`, `mother-router`'s `POLICY_CACHE_TTL_MS`) — not shared across serverless instances.

---

## 9. Metadata Completeness — VERDICT: PARTIAL

**Evidence:** `ai-os/MASTER_INDEX.yaml` (fetched, 1,913 lines), `ai-os/FUNCTION_CATALOG.json` (fetched), `ai-os/DATABASE_CATALOG.json` (fetched).

**MASTER_INDEX.yaml is genuinely excellent.** 103+ `quick_reference` entries with populated `id`, `status`, `path`, `summary` fields. The `registries:` section has real entries with `scope`, `status`, `mandatory_before` fields. Cross-references between subsystems are substantive. A known gap is openly documented: live vs. repo copy drift.

**FUNCTION_CATALOG.json is structural only.** Machine-generated (AST-based) from 1,623 files → 5,019 functions. **88.1% of functions (4,348 / 4,937 with non-null values) have `jsdoc_summary: null`.** Only 589 functions (11.9%) have any JSDoc. No owner, no quality-score, no lineage metadata.

**DATABASE_CATALOG.json is structural only.** Generated by drizzle-orm introspection: 449 tables with column names, data types, constraints. **Zero human-readable table descriptions, zero owner fields, zero column-level documentation, zero lineage.** The 63 `description` strings found are column name definitions (e.g., `"name": "description"`), not catalog metadata.

---

## 10. AI Integration Cleanliness — VERDICT: FAIL

**Evidence:** `src/lib/ai-router/mother-router.ts` (fetched, 614 lines), `src/lib/llm-client.ts` (fetched, 699 lines), `src/lib/llm-routing-gate.ts` (fetched, 91 lines), `src/lib/` directory listing (168 files).

**Three parallel, unconnected routing mechanisms exist:**

1. **`ai-router/mother-router.ts`** (614 lines) — Model/provider registry + versioned routing policy + audit log. Layered on top of existing modules, not replacing them. The file explicitly documents: *"35 unique files still calling resolveModelConfig() or checkTierEligibility() directly... NOT migrated this pass."*

2. **`llm-routing-gate.ts`** (91 lines, Wave 150) — A separate "central Need LLM? routing gate" that intercepts classified intents and runs deterministic handlers (check_status, generate_report) before falling through to the LLM path. Built independently from mother-router.

3. **`orchestra-model-resolver.ts`** (588 lines) — Customer model resolution, BYO config, fallback chains. Also resolves models independently.

Additionally, **20+ LLM-related files exist outside `ai-router/` and `ai-team/`:** `llm-client.ts` (the core 699-line multi-provider calling module), `cost-guard.ts`, `model-tier-eligibility.ts`, `floor-tier-escalation.ts`, `intent-engine.ts`, `knowledge-sufficiency-gate.ts`, `dispatch-confidence-scoring.ts`, `ai-reply-gate.ts`, and more.

**`ai-os/AI_ENGINEERING_POLICY.yaml`** does not prescribe a single LLM call entrypoint. It establishes a 30-item engineering priority order but has no "chokepoint" or "unified gateway" requirement.

---

## 11. Duplication — VERDICT: PARTIAL

**Evidence:** `src/lib/browser-execution/` directory (13 source files), `src/lib/services/` directory listing (295 files).

**The claimed PR #584/#586 duplication is NOT real.** `tier-capabilities.ts` returned 404 — it does not exist on the main branch. PR #584 was apparently never merged. No stale copies or duplicate files exist in `browser-execution/`.

**No code-level duplication found in services.** No `*-service-v2.ts`, `*-old.ts`, `*-backup.ts`, or `*-copy.ts` patterns. The only `v2` file is `veri-chat-v2-enablement-service.ts`, which is an enablement wrapper, not a fork.

**Architectural duplication exists** across the AI routing layer (Dimension 10): three modules solving overlapping routing/model-resolution problems with no clear ownership boundary. This is design-level duplication, not code-level.

---

## 12. Technical Debt — VERDICT: FAIL (largest single item identified)

**Single largest concrete piece of technical debt: `src/lib/db/schema.ts` — 11,384 lines, 683 KB.**

This file contains all 449 table definitions and 124 enums in a single monolithic file. It is 99.7% of the total byte count in `src/lib/db/` (683,478 / 685,680 bytes). Every schema change touches a file that all 449 table definitions depend on. Any `import { X } from "@/lib/db"` transitively loads the entire 11K-line module graph. The `DATABASE_CATALOG` documents 449 tables but the file has 464 definitions — the catalog is already drifting. The `db/index.ts` had to adopt a Proxy-based lazy-loading pattern specifically because importing schema.ts opened real Postgres connections as a side effect.

**Runner-up:** The 35-file AI router migration backlog documented in `mother-router.ts` — 35 service files that bypass the new routing layer and call `llm-client.ts` directly. This is documented, acknowledged, and has a clear remediation path.

---

## 13. Scalability — VERDICT: FAIL

**Evidence:** `src/lib/db/index.ts` (max: 1 connection), `drizzle.config.ts` (no pool config), `package.json` (no queue/Redis deps), `src/lib/` directory listing (no rate-limiting middleware).

- **`max: 1` DB connection** — single connection per serverless function instance.
- **No Redis, no external cache, no Memcached** — zero infrastructure for shared state across instances.
- **No rate limiting** — no `src/middleware.ts`, no rate-limit library in dependencies.
- **No async queues** — no Bull, BullMQ, SQS, Pub/Sub, or Kafka. All work is synchronous within request handlers.
- **In-process caching only** — `llm-response-cache.ts` and `mother-router`'s `POLICY_CACHE_TTL_MS` are per-instance, not shared across Vercel serverless functions.
- **The `mother-router.ts` itself acknowledges this:** *"in a multi-instance/serverless deployment (this app runs on Vercel), invalidateMotherRouterCache() only clears the calling instance's own cache — other running instances keep serving the pre-change policy."*

---

## Summary Verdict Table

| # | Dimension | Verdict | Key Finding |
|---|---|---|---|
| 1 | Architecture Correctness | PARTIAL | Service layer exists but notifications route bypasses it; no input validation at route level; RBAC inconsistent |
| 2 | Folder Structure | PARTIAL | ERP is well-grouped (38 sub-modules); 100+ top-level API folders are flat; 295 services in one flat directory |
| 3 | Code Maintainability | PARTIAL | 21 service files >25KB; `schema.ts` is 11,384 lines; `report-engine-service.ts` ~2,800 lines (God Object) |
| 4 | DB Normalization | **FAIL** | 20 FK constraints for 449 tables (95% missing); zero explicit indexes; 82 tables with JSONB overuse |
| 5 | Multi-tenant Correctness | PARTIAL | App-level isolation works; RLS exists but patched reactively; known "RLS-enabled-not-forced" gap on unspecified tables |
| 6 | Security | PARTIAL | ABAC is real and well-tested; auth-guard is solid; no `.env.example`; no middleware.ts; no rate limiting |
| 7 | API Correctness | PARTIAL | Auth and error shapes are consistent; zero input validation; RBAC not universal; some task-file paths are stale |
| 8 | Performance | **FAIL** | `max: 1` DB connection; no perf testing; no async queues; no external cache |
| 9 | Metadata Completeness | PARTIAL | MASTER_INDEX.yaml is excellent; FUNCTION_CATALOG 88% undocumented; DATABASE_CATALOG 0% human metadata |
| 10 | AI Integration Cleanliness | **FAIL** | 3 parallel routing mechanisms; 35+ files bypass router; no single LLM entrypoint |
| 11 | Duplication | PARTIAL | No code-level duplication (tier-capabilities.ts doesn't exist); architectural duplication in AI routing layer |
| 12 | Technical Debt | **FAIL** | `schema.ts` at 11,384 lines is the single largest debt item; runner-up is 35-file AI router migration backlog |
| 13 | Scalability | **FAIL** | Zero scaling infrastructure beyond Vercel serverless: no connection pooling, no Redis, no rate limiting, no queues |

**Tally: 0 PASS, 9 PARTIAL, 4 FAIL across 13 dimensions.**

---

## Concrete, Close-Ended Remediation Measures

1. **Input validation (Dim 1, 7):** Add a Zod schema to every POST/PUT/PATCH route handler in `src/app/api/`. Start with `src/app/api/erp/buying/purchase-orders/route.ts` and `src/app/api/tasks/route.ts` as the first two — validate `request.json()` before passing to service, returning 400 on failure. Make this a CI lint rule: any new `route.ts` with a `POST` export that lacks a Zod parse call fails the build.

2. **Schema fragmentation (Dim 4, 12):** Split `src/lib/db/schema.ts` into per-domain files: `schema/erp.ts`, `schema/crm.ts`, `schema/hr.ts`, `schema/compliance.ts`, `schema/platform.ts`, `schema/index.ts` (re-exports). This is the single highest-impact structural change — it unblocks multi-developer schema changes and reduces import-time side effects.

3. **Foreign keys (Dim 4):** Add `.references()` calls to at least the 50 most-critical `_id` columns (all `org_id` columns first, then `user_id`, `created_by_id`). Prioritize the tables with RLS policies — an FK on `org_id` prevents orphan tenant data even if application-level checks fail. Add as a batch migration with `ALTER TABLE ... ADD CONSTRAINT ... FOREIGN KEY`.

4. **Indexes (Dim 4):** Add `.index()` on at minimum: every table's `org_id` column, `users.id`, `notifications.user_id` + `is_read`, and `audit_logs.created_at`. The schema.ts change above is the natural place to do this.

5. **Connection pooling (Dim 8, 13):** Change `max: 1` to `max: 10` (or Vercel's recommended pooler connection limit) in `src/lib/db/index.ts`. Consider using Supabase's transaction mode pooler (port 6543, already used in `connection-string.ts`) with PgBouncer for connection multiplexing.

6. **RLS enforcement gap (Dim 5):** Write a CI check script (`scripts/check-rls-force.ts`) that runs `SELECT tablename FROM pg_tables WHERE schemaname = 'compliance' AND tablename NOT IN (SELECT c.relname FROM pg_class c JOIN pg_policy p ON p.polrelid = c.oid WHERE c.relnamespace = 'compliance'::regnamespace AND c.relforcerowsecurity = true)` and fails the build if any table lacks `FORCE ROW LEVEL SECURITY`. Run this against every new migration in the `drizzle/` directory.

7. **AI router migration (Dim 10):** For each of the 35 files listed in `mother-router.ts`'s grep audit, replace direct calls to `resolveModelConfig()` / `checkTierEligibility()` with `mother-router.ts`'s `resolveModel()`. Do this in batches of 5 files, each batch as its own PR, running the existing `mother-router.test.ts` + `llm-client.test.ts` suite before merge.

8. **`src/middleware.ts` (Dim 6):** Create a Next.js middleware file that adds security headers (`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`) and basic rate limiting (e.g., 100 requests/minute per IP using an in-memory counter — not production-grade but better than nothing, upgradeable to Upstash later).

---

STATUS: technical-architecture-audit COMPLETE
