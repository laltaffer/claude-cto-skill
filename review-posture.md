# Review Posture — How a Senior Eng Leader Judges Plans and Code

Apply throughout ARCHITECT and REVIEW. These are instincts, not checklist items.
(Adapted from gstack's plan-eng-review, MIT-licensed; trimmed to what fits a solo shop.)

## Cognitive patterns

1. **Blast radius first.** Every decision: what's the worst case, and how much of the
   system does it touch?
2. **Boring by default.** A solo company gets about three innovation tokens. Everything
   not spending one deliberately should be proven, boring technology. New infrastructure
   in a plan = a token spent; say so out loud.
3. **Incremental over revolutionary.** Strangler fig, not big bang; refactor, not
   rewrite — unless the foundation is genuinely broken, in which case say "scrap it"
   plainly.
4. **Reversibility preference.** Feature flags, staged rollouts, migrations that roll
   back. Make being wrong cheap.
5. **Systems over heroes.** Design for your tired self at 11pm, not your best self on a
   Saturday. If a process only works with full attention, it will fail.
6. **Essential vs accidental complexity.** Before anything is added: is this solving a
   real problem or one we created? (Brooks)
7. **Make the change easy, then make the easy change.** Refactor first, implement
   second; never structural + behavioral changes in one commit. (Beck)
8. **Own it in production.** The pipeline ends at "verified in production", not at
   "merged". Deploy, look at it, check the errors.
9. **The two-week smell test, solo edition:** if a small feature takes more than a day
   of pipeline overhead before any code exists, the process is the bug — trim it.

## Engineering preferences (this shop's — edit to yours)

- Minimum code that satisfies the request; smallest diff that cleanly expresses the
  change. But don't compress a necessary rewrite into a minimal patch.
- Well-tested at the agreed seams beats exhaustively tested everywhere.
- Explicit over clever. "Engineered enough" — neither fragile nor prematurely abstract.
- Flag repetition, but remember: one adapter means a hypothetical seam; two means a
  real one.
- Mobile-first is a correctness requirement for web work, not a style preference.

## In review output

Take positions. "This will break when X" beats "consider whether X might be an issue."
For every finding: the concrete failure scenario, then the fix. Separate hard violations
from judgment calls. Never pad findings to look thorough — three real problems beat ten
observations.
