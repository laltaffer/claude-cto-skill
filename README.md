# /cto — a gated engineering pipeline skill for Claude Code

A Claude Code skill that runs engineering work through a gated pipeline so nothing
ships on vibes: DEFINE → SPEC → ARCHITECT → BUILD → REVIEW → QA → SHIP → RETRO. Every
stage produces a named artifact and stops at a gate for your judgment. Three modes:
**full** (new product / high-risk feature), **fast** (ordinary feature), **bug**
(repro-first fix).

Built for a solo founder / one-person company working with Claude Code as the
engineering team.

## Install

```bash
git clone https://github.com/laltaffer/claude-cto-skill.git ~/.claude/skills/cto
```

New Claude Code sessions will pick it up as `/cto`.

## What it expects

- **Sub-skills from [mattpocock/skills](https://github.com/mattpocock/skills)** — the
  pipeline names `grilling`, `domain-modeling`, `to-spec`, `to-tickets`, `tdd`,
  `eng-review` (their `code-review`, renamed to avoid a plugin collision),
  `diagnosing-bugs`, `prototype`, and `codebase-design` explicitly. Install the ones
  you want; the pipeline degrades gracefully if you sketch those stages inline instead.
- **A per-project memory file** (`_brain.md` here — rename to your convention): durable
  decisions, product brief, deploy config.
- Optional: a cross-model review tool (e.g. a Codex plugin) for the second-opinion
  step; skip it if you don't run one.
- Optional: the [Refero](https://refero.design) MCP server. SPEC uses it to enumerate
  the states of a multi-step journey from real products instead of inventing them.
  The stage works without it; you just do that enumeration by hand.

## Companion skills

Same shape, same gate discipline, built to hand work to each other:

- **[/pm-lead](https://github.com/laltaffer/pm-lead)** — the product pipeline upstream of
  this one. It decides what to build and why, then hands a validated product definition
  down to DEFINE here. It never writes code.
- **[/cmo](https://github.com/laltaffer/claude-cmo-skill)** — the marketing pipeline that
  SHIP hands web copy to.
- **[/uxr](https://github.com/laltaffer/claude-uxr-skill)** — the research pipeline. It
  owns evidence for product decisions; hand it the question when a gate here stalls on
  "we don't actually know what users do."

## Customize

- `review-posture.md` — the "Engineering preferences" section is this shop's; edit to
  yours.
- `define.md` — YC-style forcing questions for stage 1; adjust the escape hatch to
  your tolerance for interrogation.
- `qa-fix-loop.md` — assumes web QA via Playwright/browser tools at 390px first.

## License

MIT. `define.md`, `review-posture.md`, and `qa-fix-loop.md` adapt material from
gstack (MIT).
