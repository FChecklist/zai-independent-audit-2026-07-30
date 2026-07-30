# zai-independent-audit-2026-07-30

A genuinely **independent** third-party audit of VERIDIAN, performed by **Z.ai** (a separate AI
service, unrelated to the Claude Code / Claude sessions that built and self-assessed this system),
run from the Owner's own free Z.ai web-chat session (no API access assumed -- plain chat only, no
GitHub write access, no server/shell access).

## Why this repo exists (read before doing anything else)

Every prior evaluation of VERIDIAN -- gap analyses, "done" claims, maturity scores, capability
audits -- was produced by Claude Code sessions working *on* VERIDIAN. That is self-assessment, not
independent verification, and this system's own history contains multiple confirmed cases of a
self-assessment claim ("done", "live", "wired up", "proven") turning out to be false on later
re-check (see e.g. this repo's sibling analysis: a "CRM chat-wiring proven" claim that was false on
re-verify; a worker-resume prompt bug found only by re-auditing a "fixed" incident). **Do not treat
any prior claim -- in this repo, in the target repos' own docs, or anywhere else -- as true until
you have independently confirmed it with a concrete file, line, command, or live HTTP response.**

Z.ai's job is to look at VERIDIAN cold, from three specific angles, and report only what it can
back with real evidence.

## The three perspectives (one task file each, in `tasks/`)

1. **`1-owner-perspective-audit.md`** -- Is VERIDIAN actually delivering what the Owner needs:
   business completion, PROJEXA-AI.COM readiness, governance reliability?
2. **`2-deterministic-software-perspective-audit.md`** -- Is "software decides first, AI escalates
   only when needed" actually enforced end-to-end in the real code, or aspirational in places?
3. **`3-multitenant-enduser-perspective-audit.md`** -- Does the real, live system actually work
   correctly for a genuine end user in a multi-tenant setup: data isolation, real task flow, real UI?

These are independent of each other -- do them in any order, or as three separate Z.ai
conversations if that gives cleaner context per pass.

## What you can and cannot access (real, checked 2026-07-30 -- verify it still holds for you)

| Repo | Visibility | What that means for you |
|---|---|---|
| `FChecklist/compliance-tracker` | **PUBLIC** | Fully readable by anyone, no login needed. This is where most of the real application code and `ai-os/` policy docs live. |
| `FChecklist/projexa` | **PRIVATE** | You will almost certainly NOT be able to open this from a plain Z.ai browsing session. Try it first -- if you get a 404 / login wall, note that as a real access limitation in your findings rather than guessing at its contents. |
| `FChecklist/claude-control` | **PRIVATE** | Same as above -- try, then record whether you could actually read it. |
| `FChecklist/zai-independent-audit-2026-07-30` (this repo) | **PUBLIC** | You can read it. **Never put secrets, passwords, or API keys into this repo -- it is public.** |

There is **no SSH key, server login, or shell access** for you, and none will be provided -- that
is intentional, not an oversight. Anything that only exists on the live server (not committed to
a readable repo) is out of reach for you; if a task file references such a thing, treat it as an
explicit access gap to report, not something to speculate about.

**Live production URL for perspective 3:** `https://projexa-ai.com` -- confirmed reachable
(HTTP 200) as of 2026-07-30. Demo tenant login credentials exist but are deliberately **not**
written anywhere in this repo (it's public). The Owner will paste them directly into your chat
session when you reach task 3 -- if you don't have them yet, ask the Owner for them instead of
guessing or skipping the live test.

## Required methodology (apply this inside EVERY task file, not just once overall)

For each of the three task files:
1. **Run the full analysis three separate times** (three independent passes -- do not just repeat
   your first answer with different wording; re-derive each pass from the source material).
2. **Produce ONE distilled, deterministic (rules-and-logic-based) analysis** that reconciles all
   three passes -- where they agree, say so plainly; where they disagree, say so and explain why,
   and resolve it with evidence rather than picking one arbitrarily.
3. **Identify gaps** -- concrete, named, evidenced gaps between what's claimed and what you could
   verify.
4. **Suggest close-ended remediation measures** -- specific and actionable (a named file to change,
   a specific check to add, a specific test to write), never vague ("improve monitoring", "add more
   tests" is not acceptable; "add a startup-time assertion in `X.ts` that throws if `Y` is null" is).

Do not skip straight to a single-pass answer. The three-pass-then-reconcile step is the point --
it's what makes this different from a single self-assessment.

## Output format and where to save it

This repo does not pre-write any conclusions for you -- the task files give you real starting
pointers (repo names, file paths, functions, a live URL) and instruct you to verify independently,
exactly like the Claude Code sessions on this system are themselves expected to.

For each task, write ONE markdown file into the matching `findings/<perspective>/` folder (each
folder has its own short README describing the exact expected shape). Use GitHub's web "Add file"
button to create it directly on `main` -- no git command line needed, matching the existing
`zai-sap-reports-queue` convention this repo mirrors.

End every findings file with a single final line, exactly one of:
```
STATUS: <task_id> COMPLETE
STATUS: <task_id> BLOCKED -- <one-sentence reason>
```

Once a `findings/<perspective>/*.md` file exists with a `STATUS: ... COMPLETE` line, the VERIDIAN
maintainer (a Claude Code session on VERIDIAN-DEV) will review it and decide what, if anything,
gets acted on. Your job ends at producing the independent report -- not implementing fixes.

## How the Owner uses this repo (for reference)

1. Open a free Z.ai web-chat session, logged in as `raajat.agarwal@gmail.com`.
2. Paste this repo's URL and ask Z.ai to read this `README.md`, then begin
   `tasks/1-owner-perspective-audit.md`.
3. Repeat for tasks 2 and 3 (same session or a fresh one each time -- fresh sessions give more
   independence between perspectives, which is arguably better for this specific goal).
4. When Z.ai finishes a task and gives you the findings text plus a `STATUS: ... COMPLETE` line,
   copy the whole response and save it into this repo under the matching `findings/<perspective>/`
   folder via GitHub's web "Add file" button.
