---
task_id: deterministic-software-perspective-audit
priority: 2
findings_folder: findings/deterministic-software-perspective/
status: pending
---

# Task 2: Deterministic-software-perspective audit

## The question
VERIDIAN's stated design principle is: **"software decides first, AI escalates only when
needed."** In plain terms: for any given action, a deterministic rule/check in real code should
run first, and an AI/LLM call should only happen when the deterministic path genuinely cannot
decide -- not as the default path.

A same-day session on this system already investigated this question extensively (today,
2026-07-30) using its own analysis. **Your job is to independently re-verify this, using your own
reading of the real code -- not to read that session's conclusions and agree with them.** Assume
you have not seen any of that prior analysis. If you happen to encounter it, treat it as one more
unverified claim to check, not as ground truth.

## Where to actually look (real file paths, checked 2026-07-30)

All in `FChecklist/compliance-tracker` (PUBLIC repo) unless noted:

- `src/lib/services/chat-service.ts` (1,269 lines as of today) -- the chat/AI conversation layer.
  Real exported functions include `sendMessage`, `createConversation`, `detectVeriMention`,
  `detectClarificationRequest`, `resolveInstructionMismatch`, `regenerateAiReply`. Read the actual
  control flow around where a message comes in and where an AI call is triggered: is there a real
  conditional gate before the AI call, and what exactly does it check?
- `src/lib/services/capability-audit-service.ts` (775 lines as of today) -- exports
  `shouldAuditCapability`, `buildAuditPrompt`, `parseAuditVerdict`, `runCapabilityAudit`. The name
  `shouldAuditCapability` suggests a decision gate -- read its actual implementation and trace every
  call site: is it a real deterministic check, and is it actually called before every AI-audit
  invocation, or can the AI path be reached without going through it?
- `ai-os/CONSTITUTION.yaml` and `ai-os/AI_ENGINEERING_POLICY.yaml` (compliance-tracker repo) --
  the system's *stated* policy on this principle. Compare the stated policy against what the code
  in `chat-service.ts` / `capability-audit-service.ts` actually does -- do not treat the policy
  document itself as evidence that the principle is enforced.
- `ai-os/MASTER_INDEX.yaml` (compliance-tracker repo, committed copy) -- describes itself as an
  auto-generated index of every mechanism in the system, including deterministic gates/guardrails.
  Read its header/protocol section, then spot-check a few of its claimed "live" deterministic
  mechanisms against the actual source file it names -- does the named file exist, and does it do
  what the index says?

## An explicit, known access gap -- report this, don't guess around it
One component referenced in prior (unverified) analysis is a file called `gateway.py`. It does
**not** live in `compliance-tracker`. Real prompt-gateway code exists at
`scripts/prompt_gateway/gateway.py` inside `FChecklist/claude-control`, which is a **PRIVATE**
repo. You will most likely be unable to read it. If so, state plainly in your findings that any
claim about `gateway.py`'s behavior (from this repo's background notes or elsewhere) is
**unverified by you** due to lack of access, rather than describing what it "probably" does.

## What to actually determine
1. Pick at least 3 real, distinct code paths in `chat-service.ts` and/or `capability-audit-service.ts`
   where an AI/LLM call happens. For each one, trace backwards: what deterministic condition, if
   any, gates that call? Quote the actual condition (function name + line, or the literal
   conditional expression).
2. Determine whether any AI-invoking path you found runs **unconditionally** (no real gate, or a
   gate that's trivially always true) -- that would be a concrete instance of the principle being
   aspirational rather than enforced.
3. Cross-check: does `ai-os/CONSTITUTION.yaml` or `AI_ENGINEERING_POLICY.yaml` claim this principle
   is enforced at a specific named mechanism? If so, does that mechanism exist and do what's
   claimed?

## What NOT to do
- Do not accept the policy document's wording as proof of enforcement.
- Do not treat `MASTER_INDEX.yaml` marking something "live" as sufficient -- open the named file
  and confirm.
- Do not speculate about `gateway.py`'s contents if you cannot read it -- report the access gap.

## Required steps (3-pass methodology from the repo README)
1. Run this full trace-and-verify analysis three independent times, ideally re-reading the source
   files fresh each pass rather than reusing your own earlier notes.
2. Produce one distilled, reconciled analysis: where do the three passes agree the gate is real vs.
   aspirational, and where do they disagree?
3. List concrete gaps (named function, named file, named missing check).
4. Give close-ended remediation, e.g. "add an explicit early-return guard in `sendMessage` at the
   point AI is invoked, with a unit test asserting the AI call is never reached when
   `<specific deterministic condition>` is false" -- not "add more validation".

## Output
Write your findings to `findings/deterministic-software-perspective/` (see that folder's
`README.md` for the expected shape). End with
`STATUS: deterministic-software-perspective-audit COMPLETE` or
`STATUS: deterministic-software-perspective-audit BLOCKED -- <reason>`.
