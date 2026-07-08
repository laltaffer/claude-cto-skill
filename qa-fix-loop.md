# QA Fix Loop — Stage 6

Use it like a user, fix what breaks, and lock every fix down with a regression test.
(Loop adapted from gstack's /qa, MIT-licensed.)

## Setup

1. Inventory the flows this change touches (from the PRD/issues) plus the app's 2–3
   core flows regardless.
2. Test target: local dev server or preview deploy. Browser via Playwright /
   chrome-devtools tools.
3. **Mobile first.** Set viewport to 390×844 before anything else. Desktop pass second.
   Screenshots go to the project's `.scratch/` with stable names (`qa-<flow>-mobile.png`).

## The pass

For each flow: walk it as a user would — click, type, submit, navigate back. Watch for:
- Console errors (zero tolerance on the happy path)
- Layout breaks at 390px: overflow, wrapped display lines, unreachable nav
- Dead links, broken images, missing states (empty, loading, error)
- Data that doesn't round-trip (submit → reload → still there?)

## Per-bug fix loop

For every bug found, in order — never batch:

1. **Locate** the source (grep the error, trace the component).
2. **Fix** minimally.
3. **Commit** atomically with a message naming the symptom.
4. **Re-test** the exact flow that failed, same viewport.
5. **Regression test.** Write the test that would have caught it, at the right seam
   (see the `tdd` skill on seams). If no good seam exists, that's a finding — note it.

## Self-regulation

After 3 fix attempts on the same bug without progress: stop, switch to the
`diagnosing-bugs` skill (the loop you need is a repro loop, not more attempts).
If fixes start touching files unrelated to the symptom, stop and reassess scope.

## Done

- All inventoried flows pass at 390px and desktop.
- Zero console errors on happy paths.
- Every fix has a regression test (or a documented missing-seam finding).
- QA report: what was tested, what broke, what was fixed, what's deferred.
- Nothing stray landed in the repo root or home folder.
