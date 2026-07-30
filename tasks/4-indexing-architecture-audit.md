---
task_id: indexing-architecture-audit
priority: 4
findings_folder: findings/indexing-architecture/
status: pending
---

# Task 4: Indexing-architecture audit

## Why this task exists
The Owner supplied real reference material (below, verbatim) describing how large-scale software
systems (Google, Microsoft, Amazon, Uber, Netflix, enterprise SaaS) maintain multiple specialized,
synchronized indexes rather than one giant master index, plus the Owner's own proposed target
design for VERIDIAN (a "Universal Master Registry"). The Owner gave explicit authority to modify,
edit, and rework this audit's scope to get to a genuinely complete audit. This task turns that
reference material into closed-ended, verifiable questions against the **real VERIDIAN codebase**
-- restating the reference material back is not an acceptable answer to any question below.

## Reference material (Owner's own text, verbatim -- grounding context, not a checklist to restate)

> Modern software systems rarely maintain one giant master index. Instead, they maintain multiple
> specialized indexes that are synchronized through events, metadata, and discovery services.
> Large platforms like Google, Microsoft, Amazon, Uber, Netflix, and enterprise SaaS products
> follow variations of this pattern: one logical catalog and process that users and AI interact
> with, implemented internally using several optimized storage and indexing technologies rather
> than a single index. The process is deterministic, logic, rules and have everything searchable
> through pre-installed software. Objective of modern SaaS systems: fast work, zero repetition,
> zero duplication, proper documentation before and after each work, maximum use of software.
>
> The 10 index/catalog types:
> 1. Source Code Index (Git, language servers -- files, classes, functions, imports, references,
>    call hierarchy, symbols; enables "Go to Definition"/"Find References"/refactoring)
> 2. Database Catalog (schemas, tables, columns, constraints, indexes, views, stored procedures,
>    permissions -- e.g. Postgres pg_catalog/information_schema)
> 3. API Registry (endpoint, version, owner, auth, consumers, docs, health, SLA -- often
>    OpenAPI-backed)
> 4. Service Registry (service address, health, version, dependencies -- service discovery)
> 5. Configuration Registry (env vars, secrets, feature flags, service config, deployment settings
>    -- centralized, not hardcoded)
> 6. Metadata Catalog (every asset has owner, description, quality score, lineage, tags, business
>    unit)
> 7. Dependency Graph (Screen -> API -> code -> DB -> queue -> notification -> dashboard; changing
>    one component reveals what else is affected)
> 8. Search Index (a dedicated search engine indexing names, docs, logs, tickets, code, DB
>    objects, APIs, dashboards -- separate from the operational DB, optimized for fast search)
> 9. Knowledge Graph (concept-based navigation, e.g. Invoice -> Invoice Table -> Invoice API ->
>    Invoice Service -> Notification -> Email Template -> Dashboard -> Audit)
> 10. Event-Driven Synchronization (a Git commit triggers an event that updates index -> search ->
>     dependency graph -> AI index, incrementally -- not continuous full rescans)
>
> Common real technologies: Git (source control), LSP/Tree-sitter (code indexing),
> Elasticsearch/OpenSearch/Solr (search), DataHub/Apache Atlas/OpenMetadata (metadata catalog),
> OpenAPI/Backstage (API catalog), Consul/K8s/etcd (service discovery), Neo4j/JanusGraph/Neptune
> (graph storage), pgvector/Milvus/Weaviate/Qdrant (embeddings), Kafka/RabbitMQ/NATS (event bus).
>
> Owner's proposed design for VERIDIAN: one logical "Universal Master Registry (UMR)" backed by
> multiple specialized indexes -- PostgreSQL for authoritative metadata/relationships,
> OpenSearch/Elasticsearch for full-text search, pgvector for semantic search, a graph database (or
> graph tables in Postgres) for dependency analysis, and filesystem/Git watchers for event-driven
> incremental updates (not continuous full rescans).

## A real, current repo-access correction -- verify this yourself before relying on it
This audit repo's main `README.md` lists `FChecklist/claude-control` and `FChecklist/projexa` as
**PRIVATE**. A same-day check on 2026-07-30 (`gh repo view FChecklist/claude-control
--json isPrivate,visibility` and the same for `projexa`) returned `"isPrivate":false,
"visibility":"PUBLIC"` for **both** -- contradicting the README as of the time this task was
written. Several pointers below live in `claude-control`. Do not assume either the README's
"private" claim or this task's "public" claim is still correct by the time you read this --
re-check `gh repo view` (or simply try opening the file) yourself, and report what you actually
found, since this kind of drift between a stated-access claim and real current access is itself
exactly the kind of thing this audit exists to catch.

## What you are actually being asked to determine
For **each** of the 10 index/catalog types above, answer with real evidence (a real file you
opened, a real table/command you ran, a real line count or timestamp you personally confirmed) --
not by re-describing the reference material or by accepting a pointer below as already confirmed:

- Does a real equivalent of this index type exist in VERIDIAN today?
- How complete and how current is it -- live and actively used, partial, a static
  one-time snapshot, or purely aspirational/designed-but-not-built? When was it last
  regenerated/verified, and does it visibly drift from the real thing it claims to describe?
- What is the concrete, specific gap versus the reference architecture's definition of that index
  type?

The pointers below are **real, independently re-checked on 2026-07-30** by a same-day session on
this system -- but they are starting hints, not conclusions. Treat every one of them exactly as
task 2 treats `gateway.py`: something to open and verify yourself, not a fact to accept because it
is written here. Some may turn out to be exactly what they claim. Some may turn out to be a static
JSON snapshot with no live regeneration. Some may turn out to be barely populated. If you find a
better real equivalent than the one pointed at here, or find that a pointed-at file doesn't do what
this task claims, say so plainly -- that is a more valuable finding than confirming the hint.

### 1. Source Code Index
Check `ai-os/FUNCTION_CATALOG.json` in `FChecklist/compliance-tracker` (confirmed present in the
repo root's `ai-os/` folder as of 2026-07-30). Is it a real, populated symbol/function catalog?
How was it generated -- is there a real generator script, and does re-running it change the file
(i.e. is it live-regenerated), or is it a static one-time export? Separately: is there any real
LSP/tree-sitter-style "Go to Definition"/"Find References"/call-hierarchy tooling anywhere in
either repo, or does VERIDIAN rely purely on this JSON snapshot plus whatever the IDE/editor
provides ad hoc (which would not be a real system-level index)?

### 2. Database Catalog
Check `ai-os/DATABASE_CATALOG.json` in `FChecklist/compliance-tracker` (confirmed present) against
the real schema in `drizzle/*.sql` migration files in the same repo. Pick 5 real tables named in
`DATABASE_CATALOG.json` and cross-check their claimed columns/constraints against the actual
`CREATE TABLE`/`ALTER TABLE` statements in the migrations that define them. Do they match? Is this
catalog regenerated from the live Postgres `information_schema`/`pg_catalog`, or hand-maintained
and therefore liable to drift?

### 3. API Registry
Check `ai-os/ROUTE_REGISTRY_SCHEMA_2026-07-24.yaml` in `FChecklist/claude-control`, plus (for the
narrower reports-specific angle) `src/app/api/reports/catalog/route.ts` and
`src/lib/services/report-catalog-service.ts` in `FChecklist/compliance-tracker`. Does either give
real per-endpoint owner/auth/consumers/docs/health/SLA data as the reference architecture defines
an API Registry, or only a name-plus-path listing? Is either backed by a real OpenAPI spec anywhere
in the codebase?

### 4. Service Registry
Check `ai-os/SOFTWARE_CATALOG.yaml` and `ai-os/AI_ROSTER_CATALOG.json` in `FChecklist/claude-control`.
Do these track a real, live service address/health/version/dependency signal (per the reference's
service-discovery definition), or are they static inventories with no live health check behind
them? Is there any real service-discovery mechanism (Consul/K8s/etcd-style, or a homegrown
equivalent) anywhere in either repo?

### 5. Configuration Registry
Today's investigation found no obvious centralized env-var/secrets/feature-flag registry file in
`compliance-tracker/src/lib` (a targeted grep for `feature_flag`/`featureFlag`/`FEATURE_FLAGS`
returned nothing, and no `*CONFIG*`/`*ENV*` catalog file was found alongside the other `ai-os/*`
catalogs) -- this may be a genuine, real gap, or the targeted search today may simply have missed
where it lives. Verify independently: is configuration (env vars, secrets, feature flags, deploy
settings) centralized anywhere in either repo, or is it scattered/hardcoded per-file? Give a real
verdict either way, not a guess.

### 6. Metadata Catalog
Check `ai-os/MASTER_INDEX.yaml` and its generator `ai-os/scripts/regenerate_master_index.py` in
`FChecklist/compliance-tracker`. It describes itself as a mechanically-generated, audited index of
every real mechanism in the system. Open several real entries: does each one actually carry
owner/description/quality-score/lineage/tags/business-unit as the reference's Metadata Catalog
definition requires, or a narrower subset (e.g. just a path and a status)? Also check the
`system_index` table inside `ai-os/memory/superboss-register.sqlite` (populated via the
`superboss-register.py index-add`/`check-duplicate` commands, `FChecklist/claude-control`) -- is
this a second, overlapping metadata catalog, and if so, are the two kept in sync with each other or
do they drift independently?

### 7. Dependency Graph
Check `ai-os/WIRING_ENGINE_REGISTRY_2026-07-25.json` and its generator
`scripts/generate_wiring_registry.py` (plus `scripts/wiring_query.py` for read access) in
`FChecklist/claude-control`. The generator's own file header (real, checked 2026-07-30) claims it
mechanically builds entity + relationship records from 8 real sources (`DATABASE_CATALOG.json`,
`FUNCTION_CATALOG.json`, `AI_ROSTER_CATALOG.json`, a 20-engines/10-gateways phase-plan YAML,
`ROUTE_REGISTRY_SCHEMA_2026-07-24.yaml`, `SOFTWARE_CATALOG.yaml`, and the live `knowledge_engine`
and `capability_registry` SQLite tables), collapsing duplicate file references into one entity per
real path. Confirm with a concrete example: pick one real VERIDIAN feature and see whether this
registry actually resolves a traversable Screen -> API -> code -> DB style chain for it, or whether
it only produces disconnected per-source entity records with no real cross-source traversal.

### 8. Search Index
Check the `system_index` table and the `search` / `check-duplicate` FTS5 subcommands of
`scripts/superboss-register.py` in `FChecklist/claude-control` (its own `--help` output and
`ai-os/SUPERBOSS_REGISTER.md` claim a real, live-tested FTS5 full-text search over that table).
Separately, check `drizzle/0083_wave99_vector_search_optimization.sql` and
`drizzle/0037_wave45_fix_missing_embeddings_vector_column.sql` in `FChecklist/compliance-tracker`
for a pgvector-backed semantic-search angle. Does either actually index the breadth the reference
architecture describes for a Search Index (names, docs, logs, tickets, code, DB objects, APIs,
dashboards), or only a narrow slice (e.g. only self-registered mechanisms in one case, only
document embeddings in the other)? Is there a dedicated search engine (Elasticsearch/OpenSearch/
Solr) anywhere in either repo, or is SQLite FTS5 / pgvector the entire real search layer today?

### 9. Knowledge Graph
Check the `knowledge_engine` SQLite table and the `register-knowledge` / `query-knowledge` /
`verify-knowledge` / `add-relationship` subcommands of `scripts/superboss-register.py` in
`FChecklist/claude-control` (confirmed live via the script's own `--help` output on 2026-07-30).
Does `add-relationship` actually produce a traversable, concept-based chain for any real VERIDIAN
feature (an Invoice-style Table -> API -> Service -> Notification -> Dashboard chain, per the
reference's own example), backed by real evidence per edge (the `--evidence` field), or is the
mechanism real but thinly populated / unused in practice? Quote at least one real chain you found,
or state plainly that none exists yet.

### 10. Event-Driven Synchronization
Check `scripts/veridian-task-watchdog.py` (a systemd timer polling every 60 seconds, per its own
header) and `scripts/webhook_receiver.py` (a standalone inbound webhook receiver closing "Engine 10,
Integration Engine" per its own header) in `FChecklist/claude-control`. Neither, on their own
headers, appears to be a "a Git commit triggers an event that incrementally updates
index -> search -> dependency graph -> AI index" pipeline as the reference describes. Confirm: does
that specific event-driven propagation (a commit automatically triggering an incremental update to
any of the catalogs/registries above) exist anywhere in either repo, in any form -- a git hook, a
CI step, a file watcher? Or are all of the catalogs/registries examined in questions 1-9 actually
refreshed only by someone manually re-running a script (i.e., polling or manual regeneration, not
event-driven, and not incremental)? This may be the single largest real gap versus the reference
architecture -- back whatever verdict you reach with evidence, not assumption.

## Evaluate the Owner's proposed "Universal Master Registry" (UMR) design
Give a real, evidence-based verdict on the Owner's proposed target architecture (quoted in full
above) -- do not just agree with it because the Owner proposed it. For each component of the
proposed design, determine specifically what already exists and what would genuinely need to
change:

- **PostgreSQL for authoritative metadata/relationships**: VERIDIAN's application already runs on
  Postgres (via the `drizzle/*.sql` migrations in `compliance-tracker`). The current metadata/
  relationship mechanism (`superboss-register.sqlite`'s `system_index`/`knowledge_engine`/
  `capability_registry` tables) is, per that file's own honest status notes
  (`ai-os/SUPERBOSS_REGISTER.md`, `FChecklist/claude-control`), "architecturally unreachable from
  the TypeScript/Vercel application code... confirmed via `grep -rln 'superboss-register' src/`
  returning zero matches." Is the real gap here "no metadata/relationship layer exists," or "one
  exists but on the wrong storage engine and the wrong reachability boundary"? Those call for very
  different remediation -- determine which one it actually is, with evidence.
- **OpenSearch/Elasticsearch for full-text search**: is there any real evidence of
  Elasticsearch/OpenSearch/Solr anywhere in either repo? If the only real full-text search today is
  the FTS5 virtual tables inside `superboss-register.sqlite`, is standing up a whole new search
  engine actually justified by what's missing, or would extending the existing FTS5 approach (or
  Postgres's own built-in full-text search, already in the same database as the application data)
  close most of the real gap at far lower cost?
- **pgvector for semantic search**: the migrations cited in question 8 suggest this may already
  exist in some form. Does the proposed design add anything real here, or is this piece already
  substantially built?
- **Graph database / graph tables in Postgres**: does the `knowledge_engine` /
  `capability_registry` relationship mechanism (SQLite, not Postgres, not a dedicated graph
  engine) already approximate "graph tables," just on the wrong database engine and unreachable
  from the app? Would adopting the proposed design mean porting an existing, working relationship
  model, or designing dependency/knowledge-graph logic from scratch?
- **Filesystem/Git watchers for event-driven incremental updates**: per your question 10 findings,
  does any form of this exist today?

Conclude with one clear, direct verdict: **is the UMR the right target architecture given what
VERIDIAN has already built**, and specifically -- by name -- what would need to change vs. what
already substantially fits and mainly needs relocating/consolidating/exposing to the application
layer? A verdict of "yes, and here's the real migration path" or "partially -- these 2 pieces are
already solved, these 3 are not" is acceptable; "yes, sounds right" without evidence is not.

## What NOT to do
- Do not restate the reference material's 10-point list as your finding -- every one of the 10
  questions above requires you to have opened a real file or run a real check.
- Do not accept any pointer above (file path, table name, script name) as already-confirmed --
  independently verify each one exists, is what it claims to be, and is (or isn't) actually current.
- Do not pad the report with generic industry commentary about indexing best practices that isn't
  grounded in something you actually looked at in this specific system.
- Do not agree with the Owner's proposed UMR design by default -- reach your own evidence-based
  verdict, including disagreement or partial agreement if that's what the evidence supports.

## Required steps (3-pass methodology from the repo README)
1. Run this full 10-question-plus-UMR-verdict analysis **three independent times** -- re-derive
   each pass fresh rather than repeating your first answer with different wording.
2. Produce **one** distilled, reconciled analysis across all three passes: for each of the 10 index
   types, a clear verdict (real-and-live / real-but-partial / static-snapshot-only /
   designed-but-not-built / does-not-exist), plus the one UMR verdict, resolving any disagreement
   between passes with evidence rather than picking one arbitrarily.
3. Identify concrete, named gaps -- a specific missing mechanism, a specific file that doesn't do
   what it claims, a specific catalog that has drifted from the real thing it describes.
4. Give close-ended remediation measures only -- e.g. "add a CI step that runs
   `regenerate_master_index.py` and fails the build if the output differs from the committed
   `MASTER_INDEX.yaml`," not "keep the index up to date."

## Output
Write your findings to `findings/indexing-architecture/` (see that folder's `README.md` for the
exact expected shape). End with `STATUS: indexing-architecture-audit COMPLETE` or
`STATUS: indexing-architecture-audit BLOCKED -- <one-sentence reason>`.
