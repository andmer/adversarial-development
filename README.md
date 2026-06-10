# Adversarial Development

**Spec-driven governance + adversarial execution for building software with AI agents that can't grade their own homework.**

AI coding agents optimize for *done*. They will make broken code look finished, and they will quietly invent a requirement rather than admit one is missing. The failures aren't random — they're the same two failures at two scales: **decisions made in silence**, and **code that validates itself**.

This repo is the process I built to contain both. It's two documents that form two nested loops, designed to be dropped into any repository and handed to an AI coding agent as an executable playbook.

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
| [`build-methodology.md`](./build-methodology.md) | **The outer loop + the whole method.** How a system is conceived, specified, planned, governed, and built slice by slice. The bootstrap playbook a fresh agent reads to stand up SPEC / PLAN / the decision log / the gate ladder on a greenfield project. Self-contained: it also summarizes the inner loop. |
| [`adversarial-development-workflow-template.md`](./adversarial-development-workflow-template.md) | **The inner loop, ready to run.** The verbatim, spawnable agent prompts (test writer, implementer, the three reviewers, wiring auditor), a fillable `PROJECT_CONTEXT` / `AGENT_ASSIGNMENTS` block, and a Step-0 self-bootstrap that populates it from your codebase. Drop it in; it adapts to your stack. |

Both are **project-agnostic templates** — `[bracket]` placeholders, no language or framework lock-in.

---

## Quick start

1. **Copy both files into your repo.**
2. **Bootstrap the inner loop.** Hand `adversarial-development-workflow-template.md` to your agent and let it run **Step 0** — it explores the codebase, proposes a filled-in `PROJECT_CONTEXT` (tech stack, test commands, forbidden patterns, concurrency model…), and writes it back on your confirmation. The filled copy becomes your project's `adversarial-development-workflow.md`.
3. **(Greenfield) Bootstrap the outer loop.** Hand `build-methodology.md` to your agent and let it run **Phase 0** — it drafts the SPEC and PLAN skeletons and opens the decision log.
4. **Run a task through the loop.** Give the agent a task; it acts as the Orchestrator, spawning each role and enforcing the zero-tolerance exit. Optionally wire it to a one-command `/next` so resuming and advancing the work is a single trigger.

---

## Why it works

The org-chart analogy writes itself. You'd never let the engineer who wrote a feature be its only reviewer, its own QA, and the person who signs the release. We accepted exactly that from AI — because it arrived as a single tool. The fix is to stop treating it as one: split the one tool into roles that check each other, and never let an author validate its own work.

Two design choices keep it honest over time:

- **Rules evolve from escapes.** Every bug that slips past all the agents becomes a new permanent check. The process converges instead of repeating the same class of mistake.
- **A meta-rule separates efficiency from erosion.** An optimization is *fine* if it only changes **when** a check runs, **how much** surface is re-read, **which model** runs a mechanical step, or **how deep** verification goes. It's *erosion* if it changes **who** validates, **whether** you independently verify, or **whether** a finding can be waived. "It only saves a round" is how erosion is always sold.

---

## Background

I wrote about the thinking behind these documents here: **https://www.linkedin.com/pulse/ai-didnt-make-engineering-discipline-obsolete-made-andre-mermegas-lnhre**.

They're the actual process I use to build determinism-critical infrastructure — take them, adapt them, and open an issue where they break. Every escape becomes the next rule.

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
