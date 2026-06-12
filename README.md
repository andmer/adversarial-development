# Adversarial Development

**Spec-driven governance + adversarial execution for building software with AI agents that can't grade their own homework.**

AI coding agents optimize for *done*. They will make broken code look finished, and they will quietly invent a requirement rather than admit one is missing. The failures aren't random — they're the same two failures at two scales: **decisions made in silence**, and **code that validates itself**.

This repo is the process I built to contain both. Its core is two templates that form two nested loops, designed to be dropped into any repository and handed to an AI coding agent as an executable playbook — plus an operator's manual for the human running it and an annotated worked example to calibrate against.

> AI didn't make engineering discipline obsolete. It made it mandatory — at a speed where you can't catch up by hand.

---

## The two loops

**The outer loop — nothing load-bearing gets decided in silence.**
A frozen, versioned spec is the law. A dated decision log is the ADR record: every choice carries its rationale *and the alternatives it rejected*. When the agent hits an ambiguity it can't resolve, it **stops and raises a decision** — it never guesses. Undefined behavior is a process violation, not a judgment call.

**The inner loop — no agent validates its own work.**
The work is split across agents with deliberately conflicting incentives:
- a **test writer** who writes behavioral tests *before* any code and never sees the implementation;
- an **implementer** who must pass those tests and is *forbidden from weakening* them;
- **three independent reviewers** (process, domain, security) that run in parallel and never see each other's output;
- a **fresh wiring auditor** that confirms the code is actually reachable from production.

The exit condition is absolute: **every reviewer, zero findings, on the same round.** Not "mostly clean." Zero.

---

## What's in here

| File | Role |
|------|------|
| [`build-methodology.md`](./build-methodology.md) | **The outer loop + the whole method.** How a system is conceived, specified, planned, governed, and built slice by slice. The bootstrap playbook a fresh agent reads to stand up SPEC / PLAN / the decision log / the gate ladder / `CLAUDE.md` / the dedicated agent layer on a greenfield project. Self-contained: it also summarizes the inner loop. |
| [`adversarial-development-workflow-template.md`](./adversarial-development-workflow-template.md) | **The inner loop, ready to run.** The verbatim, spawnable agent prompts (test writer, implementer, the three reviewers, wiring auditor), a fillable `PROJECT_CONTEXT` / `AGENT_ASSIGNMENTS` block, and a Step-0 self-bootstrap that populates it from your codebase. Drop it in; it adapts to your stack. |
| [`OPERATOR.md`](./OPERATOR.md) | **The human's half of the protocol** — the five moments you must engage, the refusals you'll need to make, week-one calibration. |
| [`worked-example.md`](./worked-example.md) | **A real slice's journey through the loop, annotated** — reconstructed from the reference project's history so you can calibrate what normal looks like. |

The two templates are **project-agnostic** — `[bracket]` placeholders, no language or framework lock-in. `OPERATOR.md` and `worked-example.md` are companions, not templates: one is the human's half of the protocol, the other is calibration.

---

## Quick start

1. **Copy the two templates into your repo.** Then read [`OPERATOR.md`](./OPERATOR.md) — your half of the protocol — before running your first slice, and skim [`worked-example.md`](./worked-example.md) to calibrate what normal looks like.
2. **Bootstrap the inner loop.** Hand `adversarial-development-workflow-template.md` to your agent and let it run **Step 0** — it explores the codebase, proposes a filled-in `PROJECT_CONTEXT` (tech stack, test commands, forbidden patterns, concurrency model…), and writes it back on your confirmation. The filled copy becomes your project's `adversarial-development-workflow.md`.
3. **(Greenfield) Bootstrap the outer loop.** Hand `build-methodology.md` to your agent and let it run **Phase 0** — it drafts the SPEC and PLAN skeletons, opens the decision log, writes the project constitution (`CLAUDE.md`), and stands up the dedicated agent layer (a project-named test writer and implementer, each with pinned model and per-agent memory).
4. **Run a task through the loop.** Give the agent a task; it acts as the Orchestrator, spawning each role and enforcing the zero-tolerance exit. Optionally wire it to a one-command `/next` so resuming and advancing the work is a single trigger.

---

## Work decomposition — the shared vocabulary

These documents (and every artifact they generate — SPEC, PLAN, commit
messages, review audits) use one small set of identifiers. Adopt them as-is;
they are what makes a commit subject, a debt entry, or a finding citable
years later:

- **Milestone (`Mx`)** — the largest unit of scope. Only the current
  milestone is committed and task-broken-down; later milestones (`M2`,
  `M3`, …) are **demand-gated candidates** — the numbering is ordering, not
  commitment.
- **Foundation task (`Fx`)** — pre-slice, cross-cutting plumbing the
  milestone's slices depend on (scaffold, value objects, DB pool, test
  harness, determinism plumbing, …).
- **Slice (`Sx`, with letter sub-slices `Sxa`/`Sxb`/…)** — a thin
  **vertical, end-to-end-shippable** cut of milestone functionality. Each
  slice is independently spec-ratified, run through the adversarial inner
  loop, and human-signed-off before the next begins; letter suffixes carve a
  large slice into self-contained sub-deliveries.
- **Decision (`Dxx`)** — a dated row in the SPEC's **§8 decision log** (the
  ADR record): the decision, its rationale, *and the alternatives it
  rejected*, frozen by spec version. Every spec change is a versioned
  revision + a new `Dxx` row — never a silent patch.
- **Acceptance criterion (`ACn`)** — a numbered acceptance condition in the
  SPEC. Every code change traces to an `AC`, a `Dxx`, or a PLAN task — no
  orphan work.
- **Review round (`Rx`)** — one numbered iteration of the adversarial inner
  loop within a slice (e.g. `S8 R2`): implementer output → independent
  parallel reviewers → fix round. Rounds repeat until the zero-tolerance
  exit (all reviewers, zero findings, same round).
- **Finding (`FIND-<round>-<n>`)** — a single reviewer-surfaced defect
  within a round, tracked individually in the finding tracker
  (`OPEN → GENUINELY_FIXED | SHAM_FIX | NOT_ADDRESSED | PARTIALLY_FIXED |
  REBUTTED`).
- **Debt entry (`DEBT-N`)** — the stable handle for consciously-deferred
  work, held in the PLAN's carried-debt ledger. Cited by ID in commits, PRs,
  audits, and decisions — never by a prose phrase.

---

## Why it works

The org-chart analogy writes itself. You'd never let the engineer who wrote a feature be its only reviewer, its own QA, and the person who signs the release. We accepted exactly that from AI — because it arrived as a single tool. The fix is to stop treating it as one: split the one tool into roles that check each other, and never let an author validate its own work.

Two design choices keep it honest over time:

- **Rules evolve from escapes.** Every bug that slips past all the agents becomes a new permanent check. The process converges instead of repeating the same class of mistake.
- **A meta-rule separates efficiency from erosion.** An optimization is *fine* if it only changes **when** a check runs, **how much** surface is re-read, **which model** runs a mechanical step, or **how deep** verification goes. It's *erosion* if it changes **who** validates, **whether** you independently verify, or **whether** a finding can be waived. "It only saves a round" is how erosion is always sold.

---

## What it costs — and when it pays

This is not free. A full slice spawns six-plus agents — a test writer, an
implementer, two-to-three independent reviewers *per round*, and a fresh wiring
auditor — and the review rounds repeat until zero findings. That is a real token
and wall-clock multiplier over a single-agent "write it and ship it" loop. The
process is inconvenient on purpose; pretending otherwise would be the exact
editorializing the method bans.

The real numbers, mined from the reference project's git history: review rounds
per slice run a **median of ~3–4, with a maximum of 12** on the hardest slice.
What those rounds catch is the payoff. In the [worked example](./worked-example.md),
four rounds of reviewers signed off clean — and then the fresh wiring audit found
a financing adapter that was **green in every test and inert in the shipping
binary**: real values existed only behind a test-only seam, so the feature would
have shipped doing nothing, forever. That defect is invisible to a single-agent
loop. It is the class this process exists to catch.

Apply the full loop where the cost of a production bug exceeds the cost of the
process. The convergence mechanism is what makes the trade pay off over time:
every escape becomes a permanent rule, so later slices get cheaper **without
removing any guarantee** — the friction comes out, the guarantees never do. Across
~18 slices the reference project had **3 escapes slip past every agent**; each one
is now a permanent rule that catches its whole class in Round 1 by cheaper agents.

## Background

I wrote about the thinking behind these documents here: **https://www.linkedin.com/pulse/ai-didnt-make-engineering-discipline-obsolete-made-andre-mermegas-lnhre**.

They're the actual process I use to build determinism-critical infrastructure —
take them and adapt them. When an escape slips past all your agents in *your*
project, fold it into your local copy per Section G — and open an issue here with
the escape and the rule it produced. Project-specific rules stay in your copy;
generalizable ones become the next template version, recorded in
[`CHANGELOG.md`](./CHANGELOG.md). That is Section G applied across projects: the
kit converges from its users' escapes, not just its author's.

## Related

If your agent environment ships a generic engineering-skill set — e.g.
[Superpowers](https://github.com/obra/superpowers) (`spec-driven-development`,
`test-driven-development`, `code-review-and-quality`, …) — treat this kit as a
**stricter, fused specialization** of those skills: same phases, harder rules.
Where they overlap, follow this kit; reach for the generic skill for everything
it doesn't cover.

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](./LICENSE) — use it,
adapt it, build on it (even commercially); just give credit and link the license.
