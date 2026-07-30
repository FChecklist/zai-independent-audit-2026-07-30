---
task_id: distributed-intelligence-architecture-audit
priority: 5
findings_folder: findings/distributed-intelligence-architecture/
status: pending
---

# Task 5: Distributed-intelligence-architecture audit

## Why this task exists
The Owner supplied a real architectural vision (below, condensed but not reworded away from its
own meaning) proposing that the browser stop being "just a UI" and become a lightweight intelligent
execution environment -- a **"Mini VERIDIAN" / "Edge Runtime"** -- while the server remains the sole
authority for tenant isolation, security, business rules, workflow orchestration, and audit. The
Owner gave explicit authority to edit this material and make it better and close-ended. This task
turns that vision into closed-ended, verifiable questions against the **real VERIDIAN codebase** --
restating the reference material back is not an acceptable answer to any question below.

## Reference material (Owner's own vision, condensed for clarity -- grounding context, not a checklist to restate)

> "Distributed Intelligence": the browser is not just a UI, it is a lightweight intelligent
> execution environment; the server remains the authoritative system of record and governance
> layer.
>
> **Browser-side flow:** Browser UI -> Mini VERIDIAN Runtime -> Workspace/User/Tenant Context ->
> Browser Metadata Cache -> Browser Knowledge Cache -> Browser Workflow Cache -> Browser Rules
> Engine -> Browser Search -> Browser Document Viewer -> Browser Snips -> Browser AI Components ->
> Browser MCP Client -> Browser Local Agent -> Browser Intent Detection -> Browser Prompt Builder ->
> Browser Context Builder -> Browser Validation -> Browser Automation -> Browser Task Planner ->
> Browser Offline Queue -> secure request to server.
>
> **Server-side flow:** Gateway -> Authentication -> Tenant Isolation -> Universal Master Registry
> -> Business Rules -> Workflow Engine -> Policy Engine -> Execution Engine -> Database -> Search ->
> Knowledge -> Dependency Graph -> AI Router -> LLM -> Validation -> Execution -> Audit -> back to
> Browser.
>
> **Key proposed components:**
> - *Browser Metadata Engine*: screen/field/form/validation metadata, UI config, cached reference
>   data.
> - *Browser AI*: small assistive features only (rewrite, grammar, translation, summarization,
>   autocomplete, formula suggestions, search assistance, command palette, intent prediction) --
>   explicitly "assistive, should NOT make authoritative business decisions."
> - *Browser Snips*: a reusable automation library (templates, smart clipboard, saved prompts, text
>   transforms, calculations, macros).
> - *Browser MCP Client*: connects to approved external tools (calendar, email, doc editors, browser
>   automation, internal tools), gated by role/org policy.
> - *Browser Automation*: pattern-detection then local execution with **no AI and no server
>   round-trip** for repetitive form-fill, templates, table reordering, filtering, local
>   calculations.
> - *4 Intelligence Levels*: L1 UI-only (forms/tables/buttons) -> L2 Smart UI (validation/metadata/
>   search/suggestions) -> L3 Mini VERIDIAN (rules engine/snips/prompt-builder/workflow-cache/
>   browser-AI/MCP/automation/knowledge-cache) -> L4 Enterprise Connected Browser (server
>   sync/workspace context/AI context/policy updates/live metadata/task+offline+execution queues).
> - Browser responsibilities: UX, responsiveness, local assistance, cached metadata, drafts, intent
>   detection, lightweight automation, preparing complete structured requests.
> - Server responsibilities (unchanged): multi-tenant isolation, security, authorization, business
>   rules, workflow orchestration, persistent storage, AI governance, external integrations, audit,
>   compliance.

## A real, same-session correction to a starting pointer -- verify this yourself before relying on it
This task was drafted with the assumption, carried over from an earlier investigation this same
session, that VERIDIAN's `phase_5_browser_execution_tiers` design was **`not_started`**. A same-day
re-check on 2026-07-30 found that assumption to be **stale**: `compliance-tracker` PR #586
(`worker/task-20260727-065831-architecture-phase-5--browser-execution`) **merged** 2026-07-27, and a
second increment landed 2026-07-28 (commit `695d77ce`, "real NPU/Built-in-AI inference, cross-tier
storage, sync eng[ine]..."). `src/lib/browser-execution/` on `main` now contains real,
individually-tested files: `tier-detection.ts`, `tier-orchestrator.ts`, `client-compile.ts`,
`builtin-ai-engine.ts`, `npu-engine.ts`, `webllm-engine.ts`, `transformers-engine.ts`,
`worker-pool.ts`, `model-cache.ts`, `sync-engine.ts`, `cross-tier-storage.ts`, `tool-calling.ts`
(each with a matching `.test.ts`). It is also **wired into a real, live UI path**:
`src/components/veri-chat/VeriComposer.tsx` imports `compileInBrowser` from `client-compile.ts` and
calls it inside `runBrowserFirstPass()`, POSTing the resulting `{tier, fallbackChain, compileMs}` to
`/api/prompt-compiler/execute` as a best-effort enrichment on every real chat send. Status docs
exist at `ai-os/BROWSER_EXECUTION_TIERS_INCREMENT_2_STATUS_2026-07-27.md` and
`ai-os/BROWSER_LITE_LLM_TECH_DECISION_2026-07-27.md` in `compliance-tracker`, citing a phase plan at
`ai-os/VERIDIAN_ARCHITECTURE_V2_PHASE_PLAN_2026-07-25.yaml` in `claude-control` -- verify that last
file yourself, it was not independently opened before this task was written.

Separately, a **still-open, unmerged** PR #584
(`worker/task-20260726-171420-phase5-browser-execution`) exists on the same repo, touching a
*different* set of filenames for what looks like overlapping scope (`tier-capabilities.ts`,
`tier-runners.ts`, `storage-cache.ts`, `sync-queue.ts`, `model-selection.ts`, `function-tools.ts`,
`engine.ts`) -- a real, concrete possible-duplication situation worth checking on its own.

**Do not take any of the above as settled** -- it was true as observed today, 2026-07-30, on
`main`. Re-confirm it yourself (branch/PR state, file contents, and whether `VeriComposer.tsx` still
calls `compileInBrowser`) before citing it, exactly as this repo's methodology requires for every
prior claim.

**Important scope distinction, do not conflate the two:** `phase_5_browser_execution_tiers` is about
*which local compute tier runs a model* (NPU / Built-in-AI / WebLLM Lite-LLM / Transformers.js) and
a narrow "compile a prompt draft, detect a tier" first pass. It is **not** the same thing as the
Owner's "Mini VERIDIAN" vision above (metadata cache, knowledge cache, workflow cache, rules engine,
snips library, MCP client to external tools, local automation). Determine independently how much, if
any, real overlap exists -- do not assume phase_5 "counts as" Mini VERIDIAN just because both live in
`src/lib/browser-execution/`.

## Other real starting pointers (independently re-checked 2026-07-30 -- still verify them yourself)
- `src/lib/browser-intent-cache.ts` (`compliance-tracker`) -- a real, shipped, IndexedDB-based
  cache, per its own header comment "client-only recall of the user's own past VeriComposer
  submissions (mode pill + chain path + chat text)". This is workflow-**recall**, not a reusable
  template/metadata/rules library -- confirm this scope boundary yourself rather than assuming the
  name implies more than the file actually does.
- `src/app/litert-spike/` and `src/app/litert-spike-embeddings/` (`compliance-tracker`) -- per their
  own file headers, both are explicitly-marked experimental spikes ("EXPERIMENTAL SPIKE -- NOT A
  PRODUCT PAGE... touches no production file, DB table, or API route"), proving out LiteRT.js WASM
  inference for a MobileNetV2 image classifier. No input-UI, chat, or Mini-VERIDIAN-runtime code of
  any kind. Confirm they are still this narrow and still unwired from any real product surface.

## What you are actually being asked to determine

For **each** of the following proposed components, answer with real evidence (a real file you
opened, a real import/usage site, a real absence you confirmed by search) -- not by re-describing
the reference material or accepting a pointer above as already confirmed:

1. **Browser Metadata Cache** -- does a real browser-side (client-executed, e.g. IndexedDB/
   localStorage/in-memory-in-the-browser-tab) cache of screen/field/form/validation metadata or UI
   config exist? (Note: `src/lib/services/asset-registry-cache.ts` is a real "compiled metadata
   cache" per its own header -- check whether it runs in the browser or is an in-process cache on
   the **server**; these are not the same thing.)
2. **Browser Knowledge Cache** -- any real client-side cache of knowledge-graph-style data?
3. **Browser Workflow Cache** -- any real client-side cache of workflow/process state, distinct from
   `browser-intent-cache.ts`'s narrow past-submission recall?
4. **Browser Rules Engine** -- any real client-side rules evaluation? (Note:
   `src/lib/monitors/rule-engine-monitor.ts` exists -- check whether it runs server-side against the
   tenant database, which would make it a **server** rules engine, not a browser one.)
5. **Browser Snips** (reusable automation library: templates, saved prompts, text transforms,
   macros) -- any real equivalent in `src/components`, `src/hooks`, or `src/lib`?
6. **Browser AI Components** (rewrite/grammar/translation/summarization/autocomplete/formula
   suggestions/search assistance/command palette/intent prediction, explicitly non-authoritative) --
   which of these, if any, are real and shipped today, versus designed-only?
7. **Browser MCP Client** (connecting outward to approved external tools: calendar, email, doc
   editors) -- any real equivalent? (Note: `src/lib/browser-execution/tool-calling.ts` exists, but
   its own header explicitly scopes it as browser-local-model-calls-local-tool-registry, and
   explicitly distinguishes itself from `src/app/api/mcp/route.ts`, which is a **server-side MCP
   server** exposing this app's data *outward* to external MCP clients -- the reverse direction from
   what the Owner's proposal describes. Confirm whether any real browser-side client connecting out
   to calendar/email/doc-editor tools exists anywhere.)
8. **Browser Automation** (local pattern-detection execution, explicitly no AI and no server
   round-trip, for repetitive form-fill/templates/table reordering/filtering/local calculations) --
   any real equivalent? `src/hooks/` is a real, small, enumerable directory -- open it and say
   exactly what is and is not there.
9. **Browser Intent Detection / Prompt Builder** -- `client-compile.ts`'s `compileInBrowser()` (see
   the correction above) is a real, live, narrow candidate. Does it actually do intent detection and
   prompt building as the Owner's proposal describes, or a narrower single-field "detect a tier and
   pass text through" step? Be specific about what it does and does not do.

For every one of the 9 items above, give one of these verdicts, named explicitly: **real and live**,
**real but narrow/partial** (state the real scope and the real gap), or **100% design-only / does
not exist** (state what you searched and found nothing). Cite the real file path (or the real
absence of one) for each verdict -- accepting a claim above without opening the file yourself is not
an acceptable answer.

## Evaluate the 4 Intelligence Levels against reality
Which level -- L1 UI-only, L2 Smart UI, L3 Mini VERIDIAN, or L4 Enterprise Connected Browser -- does
VERIDIAN's real, shipped browser code actually reach today, if any? This session's own investigation
suggests somewhere between L1 and L2 (some real client-side validation/metadata/suggestion behavior,
per the file evidence in this task, but no rules engine/snips/MCP-client/workflow-cache to justify
L3) -- **do not just accept this**; verify independently against the real component-by-component
findings above and state your own evidence-based level assessment, including if you disagree.

## Is "Distributed Intelligence" the right target architecture?
VERIDIAN's own stated operating principle (see `ai-os/CONSTITUTION.yaml` in `compliance-tracker`,
and this repo's task 2) is "software decides first, AI escalates only when needed." Evaluate,
with reasoning grounded in what you actually found above, not agreement-by-default:

- Does pushing intelligence into the browser (metadata/knowledge/workflow caches, a rules engine, an
  automation library, browser-AI, an MCP client) **help** enforce "software decides first" -- e.g. by
  keeping more deterministic decisions client-side and cheap -- or does it **complicate** it by
  creating a second place (browser + server) where rules/decisions can drift out of sync, be
  duplicated, or disagree?
- Given what you found is real vs. design-only, is "Distributed Intelligence" (the full L3/L4 Mini
  VERIDIAN vision) currently the right next target, or does the real evidence point to a narrower,
  more achievable subset being the actual right target right now? Give a direct verdict, not a
  hedge.

## What NOT to do
- Do not restate the reference material's flow diagram or component list as your finding -- every
  question above requires you to have opened a real file, checked a real import/usage site, or run
  a real targeted search that found nothing.
- Do not accept the "same-session correction" pointer above (or any other pointer in this task) as
  already-confirmed -- independently re-verify branch/PR/merge state and file contents yourself.
- Do not conflate `phase_5_browser_execution_tiers` (real, AI-compute-tier-focused) with the full
  Mini VERIDIAN vision (metadata/knowledge/workflow caches, rules engine, snips, MCP client,
  automation) -- they overlap only partially, if at all; determine the real overlap yourself.
- Do not agree with the Owner's "Distributed Intelligence" vision by default -- reach your own
  evidence-based verdict, including disagreement or a narrower recommended scope if that is what the
  evidence supports.

## Required steps (3-pass methodology from the repo README)
1. Run this full 9-component-plus-levels-plus-verdict analysis **three independent times** --
   re-derive each pass fresh rather than repeating your first answer with different wording.
2. Produce **one** distilled, reconciled analysis across all three passes: for each of the 9
   components, one clear verdict (real-and-live / real-but-narrow-partial / design-only-does-not-
   exist) with evidence; one clear Intelligence-Level verdict; one clear "right target architecture"
   verdict -- resolving any disagreement between passes with evidence rather than picking one
   arbitrarily.
3. Identify concrete, named gaps -- a specific missing mechanism, a specific file that does less
   than its name implies, a specific duplication (e.g. the PR #584 vs PR #586 situation above, if
   still real when you check).
4. Give close-ended remediation measures only, sequenced by priority -- e.g. "given that
   `client-compile.ts` and `browser-intent-cache.ts` already exist and are wired into
   `VeriComposer.tsx`, extend `browser-intent-cache.ts`'s IndexedDB schema to store reusable
   templates (not just past submissions) before building any new Browser Snips module from
   scratch," not "build out the browser layer." State explicitly **which single browser-side
   capability would deliver the most real value first**, given what already exists to build on, and
   why.

## Output
Write your findings to `findings/distributed-intelligence-architecture/` (see that folder's
`README.md` for the exact expected shape). End with
`STATUS: distributed-intelligence-architecture-audit COMPLETE` or
`STATUS: distributed-intelligence-architecture-audit BLOCKED -- <one-sentence reason>`.
