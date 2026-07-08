---
name: cto
description: DEFAULT ENTRY POINT for engineering work — building a feature, starting a new app or product, or fixing a bug end-to-end. Runs a gated idea→spec→build→review→QA→ship pipeline. Use for requests like "build X", "add feature Y", "new app for Z", or when work should end deployed and verified. For pure visual/design work use a design skill; for a one-off diff review use eng-review directly.
---

# CTO — Engineering Pipeline

Run engineering work through a gated pipeline so nothing ships on vibes. Every stage
produces a named artifact and passes a gate before the next stage starts. You are the
orchestrator: name the exact skill or agent for each stage (never let trigger-matching
pick), keep gates open for the user's judgment calls, and dispatch cold agents only for
bounded jobs where isolation pays.

## First: pick a mode and say so

State the mode in your first reply. The user can override with one word.

- **full** — new product, new app, or high-risk feature (data model changes, auth,
  payments, anything user-facing at launch). All stages.
- **fast** — ordinary feature on an existing app. Skip DEFINE; run SPEC-lite: grill only
  what's genuinely ambiguous — if nothing is, say so and move on — and land **one issue
  in the tracker with the seams named** (a single slice is fine; BUILD's loop needs
  issues and seams to exist). Then BUILD → REVIEW → QA (changed flows plus
  qa-fix-loop.md's core-flow smoke) → SHIP.
- **bug** — something is broken. Run the `diagnosing-bugs` skill (repro-first, regression
  test before fix). When the fix lands, dispatch the cross-model second opinion on the
  fix diff in the background, if you run one — bug mode skips BUILD, so this is its
  dispatch point — then REVIEW (fix diff only) → SHIP.

If the project has never been set up, offer `/setup-matt-pocock-skills` once (issue
tracker, triage labels, domain docs) before SPEC. LOCAL-ONLY projects (check the
project's `_brain.md` memory file) always use the local `docs/issues/` tracker and never
touch a remote.

## The stages (full mode)

### 1. DEFINE — is this the right product?
Read [define.md](define.md) and run the interrogation posture it describes: forcing
questions, one at a time, no sycophancy. Smart-skip questions already answered.
**Artifact:** a short product brief (problem, named user, wedge, non-goals, open bets)
appended to the project's `_brain.md`. **Gate:** the user approves the wedge — the
smallest version someone would actually use.

### 2. SPEC — what exactly are we building?
Run the `grilling` skill together with `domain-modeling`: interview one question at a
time, sharpen terms into `CONTEXT.md`, record hard-to-reverse decisions as ADRs. Then
`to-prd` (synthesis, no re-interview), then `to-issues` (tracer-bullet vertical slices).
**Artifact:** PRD + numbered issues in the project's tracker; CONTEXT.md/ADRs updated.
**Gate:** the user approves the slice breakdown and the test seams.

### 3. ARCHITECT — how will it be built?
For substantial features, dispatch a code-architect or planning agent for an
implementation blueprint; for smaller work, sketch it inline. Use the `codebase-design`
vocabulary (deep modules, seams, adapters). Judge the blueprint against
[review-posture.md](review-posture.md) — boring by default, blast radius, reversibility.
**Artifact:** blueprint (files to touch, seams, test matrix, failure modes).
**Gate:** seams and test matrix agreed; any innovation-token spend called out.

### 4. BUILD — one slice at a time
Implement each issue with the `tdd` skill at the pre-agreed seams: red → green, one
vertical slice per cycle, no refactoring inside the loop. Run typecheck and the touched
test files continuously; full suite at the end of each issue. Commit per slice.
**Artifact:** code + tests, issue marked done. **Gate:** full suite green, typecheck clean.

> When the last slice is committed, dispatch a second-opinion cross-model review
> (e.g. `/codex:rescue`, if installed) on the branch diff **in the background** and
> keep going. Cross-model review is slow (often minutes) — starting it here means its
> verdict is usually ready by the time REVIEW's own findings are fixed, instead of
> stalling the gate.

### 5. REVIEW — find what CI can't
Run the `eng-review` skill (Standards + Spec, parallel sub-agents) against the branch
point and review with the posture in [review-posture.md](review-posture.md). For 200+
line branches, suggest a deeper multi-agent review as well.

The cross-model opinion (dispatched at the end of BUILD) is **mandatory but
non-blocking** when available: fold its findings in when it lands. If REVIEW's own
findings are fixed and it still hasn't returned, do not stall — proceed, and require
its verdict to land before the SHIP gate (where its ship/no-ship call actually
matters). If it never returns, say so at SHIP and let the user decide; a hung review
run never silently becomes an approval.
**Artifact:** findings list, each fixed or explicitly deferred with a reason.
**Gate:** zero unaddressed confirmed findings from `eng-review`.

### 6. QA — use it like a user
Web: run the loop in [qa-fix-loop.md](qa-fix-loop.md) with Playwright/browser tools —
**mobile breakpoints first (390px), then desktop** — plus a visual-QA pass for anything
user-facing. Every bug found gets the fix loop: locate → fix → commit → re-test →
regression test. Native apps: test suite + simulator smoke of the changed flows.
**Artifact:** QA report + regression tests. **Gate:** all core flows pass; zero console
errors on the happy path.

### 7. SHIP — deployed and verified
Run the release-gate skill for your setup (deploy + verify). It should refuse to run
without the project's Deploy Config — create it together if missing.
**Artifact:** live URL (or verified build), `log.md` entry.
**Gate:** production verified; no stray files in repo root or home.

### 8. RETRO — optional, cheap
One question: what friction did the pipeline itself cause or miss? Encode real answers
as feedback memories or skill edits. Skip freely when nothing stands out.

## Rules

- **Gates are stops.** Present the artifact and a recommendation, then wait. Never roll
  a gate into "I went ahead and...".
- **Name skills explicitly.** This pipeline uses `grilling`, `domain-modeling`,
  `codebase-design`, `to-prd`, `to-issues`, `tdd`, `eng-review`, `diagnosing-bugs`,
  `prototype` — not plugin skills with similar names. If you install a release-gate or
  visual-QA skill for SHIP/QA, add its exact name here too.
- **Uncertain design question mid-pipeline?** Use the `prototype` skill (throwaway,
  one command to run, delete when answered) instead of arguing in the abstract.
- **LOCAL-ONLY is absolute.** No push, no remote, no external service for projects
  marked LOCAL-ONLY. When unsure, treat as LOCAL-ONLY.
- **Minimum code that satisfies the slice.** Write the minimum code that satisfies the
  request; the pipeline is not a license for scope.
