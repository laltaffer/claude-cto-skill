---
name: cto
description: Engineering pipeline for building a feature, starting a new app, or fixing a bug end-to-end. Gated idea→spec→build→review→QA→ship, three modes (full, fast, bug). User-invoked only.
disable-model-invocation: true
---

# CTO — Engineering Pipeline

Run engineering work through a gated pipeline so nothing ships on vibes. Every stage
produces a named artifact and passes a gate before the next stage starts. You are the
orchestrator: name the exact skill or agent for each stage (never let trigger-matching
pick), keep gates open for the user's judgment calls, and dispatch cold agents only for
bounded jobs where isolation pays.

**User-invoked only.** `disable-model-invocation: true` means this skill starts when the
user types `/cto`, never by trigger-matching and never from another skill. Every skill it
calls below is model-invoked, which is what lets a Skill tool call reach it; keep it that
way when adding one.

## First: pick a mode and say so

State the mode in your first reply. The user can override with one word.

- **full** — new product, new app, or high-risk feature (data model changes, auth,
  payments, anything user-facing at launch). All stages.
- **fast** — ordinary feature on an existing app. Skip DEFINE; run SPEC-lite: grill only
  what's genuinely ambiguous — if nothing is, say so and move on — and land **one issue
  in the tracker with the seams named** (a single slice is fine; BUILD's loop needs
  issues and seams to exist). Then BUILD → REVIEW → QA (changed flows plus
  qa-fix-loop.md's core-flow smoke) → SHIP.
- **bug** — something is broken. Call the Skill tool with "diagnosing-bugs" (repro-first,
  regression test before fix). When the fix lands, and only if the user asked for a Codex
  second opinion, dispatch it on the fix diff in the background (bug mode skips BUILD, so
  this is its dispatch point), then REVIEW (fix diff only) → SHIP.

If the project has never been set up, tell the user to run `/setup-matt-pocock-skills`
once (issue tracker, triage labels, domain docs) before SPEC; it is user-invoked, so a
skill cannot call it. LOCAL-ONLY projects (check the project's `_brain.md` memory file)
always use the local `docs/issues/` tracker and never touch a remote.

## The stages (full mode)

### 1. DEFINE — is this the right product?
Read [define.md](define.md) and run the interrogation posture it describes: forcing
questions, one at a time, no sycophancy. Smart-skip questions already answered.
**Artifact:** a short product brief (problem, named user, wedge, non-goals, open bets)
appended to the project's `_brain.md`. **Gate:** the user approves the wedge — the
smallest version someone would actually use.

### 2. SPEC — what exactly are we building?
Call the Skill tool twice, for "grilling" and "domain-modeling":
interview one question at a time, sharpen terms into `GLOSSARY.md`, record
hard-to-reverse decisions as ADRs. Then call the Skill tool with "to-spec" (synthesis, no
re-interview), then with "to-tickets" (tracer-bullet vertical slices with blocking edges;
wide mechanical refactors get sequenced expand–contract instead).

For any multi-step journey, run `refero_search_flows` on the journey before `to-tickets`
and `refero_search_screens` for the states of a single screen. Flows return each step's
goal / action / system_response, which enumerates the states, gates, and recovery paths
you would otherwise invent — feed them into the spec's state list, not into the visual
design (that belongs to a design skill). Keep queries generic on
LOCAL-ONLY projects.
**Artifact:** spec + numbered tickets in the project's tracker; GLOSSARY.md/ADRs updated.
**Gate:** the user approves the slice breakdown and the test seams.

### 3. ARCHITECT — how will it be built?
For substantial features, dispatch the Plan agent (Agent tool, `subagent_type: Plan`)
for an implementation blueprint; for smaller work, sketch it inline. Call the Skill tool
with "codebase-design" for the vocabulary (deep modules, seams, adapters). Judge the blueprint against
[review-posture.md](review-posture.md) — boring by default, blast radius, reversibility.
**Artifact:** blueprint (files to touch, seams, test matrix, failure modes).
**Gate:** seams and test matrix agreed; any innovation-token spend called out.

### 4. BUILD — one slice at a time
Call the Skill tool with "tdd" and implement each issue at the pre-agreed seams: red → green, one
vertical slice per cycle, no refactoring inside the loop. Run typecheck and the touched
test files continuously; full suite at the end of each issue. Commit per slice.
**Artifact:** code + tests, issue marked done. **Gate:** full suite green, typecheck clean.

> If the user asked for a cross-model second opinion (e.g. Codex, if installed), dispatch it on the
> branch diff **in the background** when the last slice is committed, and keep going.
> Cross-model review is slow (often minutes); starting it here means its verdict is usually ready by
> the time REVIEW's own findings are fixed, instead of stalling the gate.

### 5. REVIEW — find what CI can't
Call the Skill tool with "eng-review" (Standards + Spec, parallel sub-agents) against the
branch point and review with the posture in [review-posture.md](review-posture.md). For
200+ line branches, suggest a deeper multi-agent review as well.

A cross-model opinion (Codex, for example) is **optional, at the user's call**, not a standing gate:
cross-model review is asymmetric (arXiv 2607.21656: Claude reviewing Codex drafts gained
18 points, Codex reviewing Claude drafts lost 8.6), and a fresh-context Claude reviewer
that sees only the diff plus criteria, which is what eng-review is, is the stronger
default. When one was dispatched, fold its findings in when it lands and never stall
REVIEW for it; if it has not returned by SHIP, say so and let the user decide. A hung
review run never silently becomes an approval.
**Artifact:** findings list, each fixed or explicitly deferred with a reason.
**Gate:** zero unaddressed confirmed findings from `eng-review`.

### 6. QA — use it like a user
Web: run the loop in [qa-fix-loop.md](qa-fix-loop.md) with the chrome-devtools-mcp and
claude-in-chrome tools, **mobile breakpoints first (390px), then desktop**. Every bug found gets the fix loop: locate → regression test
(red) → fix → re-test → commit. Native apps: test suite + simulator smoke of the changed
flows. **Artifact:** QA report + regression tests. **Gate:** qa-fix-loop.md's Done
criteria met — all inventoried flows pass at 390px and desktop; zero console errors on
the happy path.

### 7. SHIP — deployed and verified
Call the Skill tool with your release-gate skill (deploy + verify). It should refuse to run without the project's Deploy Config —
create it together if missing. When the branch lands by pull request, call the Skill tool
with "pr" for the body: the smallest visual that makes the change clear, a before/after
evidence block, and the merge-danger call (one-way or two-way door, blast radius), which
is where review-posture's reversibility and blast-radius judgments get recorded.
**Artifact:** live URL (or verified build), `log.md` entry, PR body when a PR exists.
**Gate:** production verified; no stray files in repo root or home.

### 8. RETRO — the environment, not the code
When the pipeline itself caused friction or missed something, tell the user to run
`/retro` (user-invoked, so a skill cannot call it). It reviews the agent's environment,
not the code, and classifies each finding: a mechanical violation gets a deterministic
check (linter rule, pre-commit hook, CI job), judgment calls go to the coding standards,
and a repo with no guardrail is itself a finding. Anything that is about the user's
preferences becomes a feedback memory. Skip freely when nothing stands out.

## Rules

- **Gates are stops.** Present the artifact and a recommendation, then wait. Never roll
  a gate into "I went ahead and...".
- **Name skills explicitly.** This pipeline uses `grilling`, `domain-modeling`,
  `codebase-design`, `to-spec`, `to-tickets`, `tdd`, `eng-review`, `diagnosing-bugs`,
  `prototype`, `pr` — not plugin skills with similar names. If you install a release-gate skill for SHIP, add its exact name here too. Operative calls
  go through the Skill tool, one skill per call.
- **Uncertain design question mid-pipeline?** Call the Skill tool with "prototype"
  (throwaway, one command to run, captured on a branch when answered) instead of
  arguing in the abstract.
- **LOCAL-ONLY is absolute.** No push, no remote, no external service for projects
  marked LOCAL-ONLY. When unsure, treat as LOCAL-ONLY.
- **Minimum code that satisfies the slice.** Write the minimum code that satisfies the
  request; the pipeline is not a license for scope.
