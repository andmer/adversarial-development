# Build Methodology — Spec-Driven Genesis + Adversarial Execution (v1)

> A portable, project-agnostic playbook for building a high-assurance software
> system from scratch and keeping its design honest as it grows. Hand this file
> to Claude Code at the start of a new project. It is the **outer loop** (how a
> system is conceived, specified, planned, and governed) wrapped around the
> **inner loop** (how each unit of work is built under adversarial review). Part 2
> summarizes the inner loop so this file is self-contained; the **verbatim,
> spawnable agent prompts** for it live in the reusable drop-in companion
> **`adversarial-development-workflow-template.md`**. Copy that template into the
> project and run its Step-0 self-bootstrap to fill the `[bracket]` placeholders —
> the filled-in copy becomes the project's own `adversarial-development-workflow.md`,
> which is what the per-task loop and the `/next` command actually spawn from.

---

## For Claude Code: Read This First

When a user hands you this file ("bootstrap a new project with @build-methodology",
"follow our build methodology"), **you are the Steward of the process, not just
an implementer.** Treat this as an executable playbook, not reference reading.

Your standing responsibilities, in order:

1. **Bootstrap the governing documents if they do not exist** (Phase 0). A
   greenfield repo has no SPEC, no PLAN, no gate ladder. You create the
   skeletons, confirm them with the user, and write them in.
2. **Never make a load-bearing decision silently.** An undefined behavior is a
   **STOP** → a versioned spec revision + a dated decision-log row with
   rationale *and rejected alternatives*. This is the single most important
   rule in the document. (§ The §8 Decision Discipline.)
3. **Build only in thin vertical slices, each through the inner adversarial
   loop**, each leaving the full gate green, each an atomic signed commit, each
   ending by updating the resume pointer. (Part 2.)
4. **Run the checkpoint audits** at the named milestones — independent reviewer
   agents + human sign-off — before proceeding. (Part 3.)
5. **Evolve the rules from every escape.** A bug that slips through every agent
   is a missing rule; add it. (Part 3.)

**The process is the product. The discipline is inconvenient on purpose.** Every
shortcut your completion bias proposes is a defect that ships. If you think "this
step isn't necessary for this task," that thought *is* the completion bias the
process exists to defeat. Run the step.

---

## The Mental Model: Two Nested Loops

```
OUTER LOOP — Governance (this playbook, Parts 1 & 3)
  idea ──► SPEC (frozen, versioned, §8 decision log = ADR record)
            │
            ▼
          PLAN (Foundation + demand-gated slices; ▶ resume pointer; DEBT-N ledger)
            │
            ▼   for each slice / task:
          ┌─────────────────────────────────────────────┐
          │  INNER LOOP — Adversarial Execution (Part 2) │
          │  context → tests-first → implement → review  │
          │  (independent, parallel, zero-tolerance) →   │
          │  wiring audit → CI gate → signed commit       │
          └─────────────────────────────────────────────┘
            │
            ▼
          checkpoint audit (independent agents + human) at milestones
            │
            ▼
          evolve the rules from every escape ──► (feeds back into SPEC/PLAN/rules)
```

The two loops share one principle at two scales: **no actor validates its own
work, and nothing load-bearing is decided without a record.** The inner loop
applies it to code (the author never reviews their own code; tests are written
by someone who never sees the implementation). The outer loop applies it to
design (a decision is never made mid-build; it is a versioned, dated, rationale-
bearing record that is never silently re-litigated).

### How this maps onto generic "skills" (Superpowers / agent-skills)

If your environment ships generic engineering skills — e.g. the
[Superpowers](https://github.com/obra/superpowers) skill set
(`spec-driven-development`, `planning-and-task-breakdown`,
`incremental-implementation`, `test-driven-development`,
`code-review-and-quality`, `documentation-and-adrs`,
`git-workflow-and-versioning`) — this playbook is a **stricter, fused
specialization** of them. The instruction-priority rule is: **user/project
instructions > skills > defaults.** Where a generic skill and this playbook
overlap, follow this playbook; invoke the generic skill only for the parts this
playbook does not cover (e.g. `frontend-ui-engineering`, `security-and-hardening`
deep-dives, `browser-testing-with-devtools`). Concretely:

| Generic skill | This playbook's stricter version |
|---|---|
| `idea-refine` | Phase A — converge an idea into a frozen SPEC §1 objective + acceptance criteria |
| `spec-driven-development` | Part 1 — SPEC is **frozen + versioned**; §8 is the ADR record; SPEC wins all conflicts |
| `planning-and-task-breakdown` | Part 1 — PLAN: Foundation tasks + demand-gated vertical slices + DEBT-N ledger |
| `incremental-implementation` + `test-driven-development` + `code-review-and-quality` | Part 2 — the **adversarial inner loop**: tests-first-by-a-blind-author, zero-tolerance multi-reviewer, wiring audit |
| `documentation-and-adrs` | The §8 decision log — every Dxx row is an ADR with rationale + rejected alternatives |
| `git-workflow-and-versioning` | Trunk-based, atomic, signed commits, each leaving the full gate green |

---

## Phase 0 — Bootstrap (First Run Only)

Before any feature work, establish the governing artifacts. If they already
exist (resuming a project), skip to the Resume Protocol at the end and continue.

**Step 0.1 — Survey the ground.** Read whatever exists: any README, existing
code, dependency manifests (`go.mod`, `package.json`, `Cargo.toml`,
`pyproject.toml`, `pom.xml`), CI config, and any `CLAUDE.md`/`AGENTS.md`/
`GEMINI.md`. For a true greenfield, there is little — that is expected.

**Step 0.2 — Refine the idea into an objective (Phase A).** If the user's intent
is vague, do not guess. Diverge then converge:
- What is this, in one sentence? What is it explicitly **NOT** (permanent
  non-goals)? (e.g. "an auditable evaluation engine — never performs real-world
  side effects; never an X; never a Y." The non-goals are as load-bearing as the
  goals.)
- Who is the user/principal, and what is the trust boundary?
- What is the single hardest invariant the system must never violate? (This
  becomes the spine of the spec — e.g. *determinism*: identical inputs ⇒
  identical, re-auditable output forever; or *conservation*: a tracked total
  changes only through defined events; or a hard latency/availability bound.)
- State your assumptions explicitly and get them corrected **now**:
  ```
  ASSUMPTIONS I'M MAKING:
  1. [scope]   2. [architecture]   3. [non-goals]
  → Correct me now or I proceed with these.
  ```

**Step 0.3 — Draft the SPEC skeleton** (see Part 1 for the section layout).
Propose it to the user as a single artifact. Do not over-specify; a spec is a
contract, not a design dump. Freeze it at **v1.0** once the user signs off, and
open the **§8 decision log** with the foundational decisions (`D1` language,
`D2` runtime/data tier, etc.), each as a dated row with rationale + rejected
alternatives.

**Step 0.4 — Draft the PLAN skeleton** — Foundation tasks (`F1…Fn`) and the
first milestone's vertical slices (`S1…Sn`). Add the **▶ Status & Next** block
at the very top (the canonical resume pointer) and an empty **Carried debt
ledger** section.

**Step 0.5 — Establish the gate ladder and tooling.** Pin tool versions so
local ≡ CI. Define three gate tiers (Part 2 § Gate Ladder): a fast Docker-free
*frequent* gate, an integration *between-rounds* gate, and the *end-of-slice*
full gate. Pick one task runner and forbid reintroducing others.

**Step 0.6 — Drop in the inner-loop template and fill PROJECT_CONTEXT.** Copy
`adversarial-development-workflow-template.md` into the project and run its
**Step 0** — it self-populates `PROJECT_CONTEXT` / `AGENT_ASSIGNMENTS` from the
codebase, proposes the filled block, and writes it back into itself on your
confirmation. This is the block every inner-loop subagent receives. Confirm the
nuanced fields (forbidden patterns, concurrency model, time/ID policy,
test-discipline invariants) with the user; flag low confidence. The filled copy
is the project's `adversarial-development-workflow.md` from here on.

**Step 0.7 — Write the project constitution (`CLAUDE.md`) and stand up the
agent crew.** Two artifacts everything downstream assumes exist:

- **`CLAUDE.md`** (or your runner's auto-loaded equivalent — `AGENTS.md`,
  `GEMINI.md`) — the entry point a cold session reads before anything else.
  The Resume Protocol (end of this document) starts from it; without it the
  cheap cold-start property is lost. Keep it short enough to load every
  session, and cover, in order: what the project is (one paragraph); the
  document hierarchy and "SPEC wins"; the resume protocol (numbered, exact);
  the commands — the gate ladder with each tier's role spelled out (frequent
  vs between-rounds vs end-of-slice, and that "green" is defined ONLY by the
  prescribed recipes); the commit-validation rule (green from every tier) and
  the flake-triage discipline; the big-picture architecture invariants that
  take multiple files to see; the short list of hard, easy-to-violate
  invariants; and a one-line workflow summary.
- **The dedicated agent layer** — at minimum a dedicated Step 2 test writer
  and Step 3 implementer: project-named agent definitions with the model
  pinned and a per-agent memory directory each. Then **run the roster
  derivation** (Part 2 § "Deriving the project's specialist roster") against
  the freshly-frozen SPEC and propose the resulting specialist roster to the
  user in the same confirmation round as the PROJECT_CONTEXT block — create
  only what the user confirms. Record every assignment in the workflow
  file's `AGENT_ASSIGNMENTS`.

Also create the **`/next` command** now (template at the end of this document)
so the steady-state loop is one trigger from day one.

**Step 0.8 — Write it all back and commit.** The Foundation scaffold is itself
the first slice — run it through a (lightweight) inner loop, then commit signed.

After Phase 0, the repo's documents are the source of truth. Conversation memory
is not. Everything below is the steady-state loop.

---

# Part 1 — Genesis & Governance (The Outer Loop)

## The Document Hierarchy Is The Law

Establish a small, fixed set of governing documents with a strict precedence
order. A representative set (rename to taste, keep the roles):

- **`SPEC.md`** — the authoritative contract. **FROZEN and versioned.** Defines
  the system in numbered sections. Its **§8 decision log (dated `Dxx` rows) is
  the ADR record** — rationale + rejected alternatives live there. Changing
  scope or a decision is an explicit **versioned revision + a new dated row**,
  never a silent mid-build patch.
- **`PLAN.md`** — task breakdown for the current milestone (Foundation + slices).
  Later milestones are **demand-gated, not pre-committed.** Holds the **▶ Status
  & Next** resume pointer and the **DEBT-N ledger**.
- **`ARCHITECTURE.md`** — a *derived*, diagrammed map. Non-authoritative; a
  reading aid, never a source of truth.
- **`README.md`** — quick start / commands.
- **External contract** (e.g. `api/openapi.yaml`) — if partners/clients build
  against an interface, that interface is a contract with its own
  breaking-change discipline and a **drift-guard test** in CI (a handler and the
  spec disagreeing fails the build). Derived from SPEC, but a hard gate.

**If any two disagree, `SPEC.md` wins.** Every code change must trace to a SPEC
acceptance criterion / `Dxx` decision, or a PLAN task. No orphan work.

## The SPEC: Frozen, Versioned, Section-Numbered

A spec is a *contract you freeze*, not a document you continuously edit. One
workable section layout (an example — adapt the section set to your system):

```
# <System> — Specification     (status header: FROZEN vX.Y; Dn count; date)
1. Objective              — what it is, the spine invariant
2. Architecture           — the structural model (e.g. modular monolith, DDD)
3. Commands               — how it is built/run/tested
4. Project structure      — directory & boundary layout
5. Code style             — hard invariants (value-object & precision rules…)
6. Testing strategy       — what is blocking; any replay/golden/audit contract
7. Boundaries             — trust boundaries, auth model, what NEVER happens
8. Open decisions (tracked) — THE DECISION LOG / ADR record  ◄── most important
9. The domain model       — the resolved core (the central rules / state machine)
10. Scope — thin vertical slice (acceptance criteria AC1…ACn)
11. Permanent non-goals   — true NEVER, definitional
12. Milestone roadmap     — everything committed; milestones are ORDER not maybe
13. Cross-cutting / operational requirements
```

The status header makes the resume cheap: `FROZEN vX.Y (D1–Dn, <date>)`.

## The §8 Decision Discipline (The Heart of the Outer Loop)

**Every undefined load-bearing behavior is a STOP.** You do not pick a value and
move on. You raise a decision: a **versioned spec revision** that adds a dated
row to §8.

Row format (the decision log is a *tracked-decisions table* — each row is a
question with a current answer and a resolution status; once settled the answer
cell holds the resolution and the last column flips to `RESOLVED`):

```
| #  | Decision (bold topic) | Current default / resolution                       | Resolve by |
|----|-----------------------|----------------------------------------------------|------------|
| D1 | **Language**          | **Go — RESOLVED <date>.** <the decision, its        | RESOLVED   |
|    |                       | rationale, AND why-not the rejected alternative.>  |            |
```

The bold topic leads the Decision cell; the dated decision text, its rationale,
and the rejected alternatives all live together in the resolution cell; the
final column is a target date while open and flips to `RESOLVED` when settled.
Adapt the column names to taste — what is load-bearing is: a stable immutable
**ID**, the **dated decision**, its **rationale**, and the **rejected
alternatives**, all in one row that is never re-litigated.

Rules that bite:
- **Rationale + rejected alternatives are mandatory.** A row that says only
  "we chose X" is not an ADR. Future-you must be able to see *why not Y*.
- **Frozen decisions are never re-litigated.** When a new request smells like a
  decision you already made, check §8 first — the rejected alternative is often
  already recorded. Do not re-open it; cite it.
- **Spec changes are versioned revisions, never silent patches.** Bump the
  version, add the row, update the status header, in the *same commit* as the
  code that depends on the decision.
- **Pre-release, revisions are cheap.** Before anything has shipped to a user
  who depends on stability, you may freely re-record frozen artifacts (golden
  fixtures, recorded baselines) on a genuine model change — still as a versioned
  §8 revision, just without contorting to preserve backward compatibility.
- **Plain language, not legalese.** Present decisions and §8 raises in plain
  words to the user; the codename/ID trail goes in the written row, not the
  question you ask.

### The Ambiguity Sweep (raise them all at once)

Before writing any test for a slice, enumerate **every** open degree of freedom
on a load-bearing axis and raise them in **one** pre-slice questionnaire — not a
drip of follow-ups. Each ambiguity resolved pre-slice is one fewer
implement→review→fix loop later. Walk this 9-axis checklist; for each axis the
slice touches, either record the answer the spec already pins or add a question:

1. **Sign / direction** — positive vs negative, which side is which; whose perspective.
2. **Counterparty / conservation** — if the change moves a quantity, what conserves it; who is the equal-and-opposite side; what event class may legally change a total.
3. **Cold-boot / NULL-state** — first-ever run, no prior row, absent value: seed? skip? error?
4. **Error-class handling** — fail-closed-loud vs tolerated-silent per failure class; which errors abort the transaction.
5. **Cadence / when-fires** — event-driven vs scheduled; every tick vs boundary; exactly-once vs at-least-once.
6. **Wire / event grammar** — exact serialized shape, field names, ordering, versioning of any emitted record.
7. **Production data SOURCE** — does the adapter GENERATE values itself on the production path, or only test-populate? (The phantom-in-production trap — see Part 2.)
8. **Determinism inputs** (if the system has a replay/audit contract) — every input to the reproducible result: seed, clock, sequence, version, ordering.
9. **Schema / migration** — new columns/tables, nullability, defaults, backfill, forward-compatibility.

If the user defers an axis, **record the deferral explicitly** as a DEBT-N item
— never silently pick a value.

## The PLAN: Foundation, Demand-Gated Slices, Resume Pointer, Debt Ledger

**Structure.** A `## ▶ Status & Next` block at the very top, then sections:
Foundation tasks (`F1…Fn` — the cross-cutting scaffolding every slice needs),
then milestone slices (`S1…Sn` — thin vertical slices, each mapping to a SPEC
acceptance criterion). **Later milestones are demand-gated** — listed as
committed *order*, not pre-built. YAGNI is a hard rule: never architect toward a
hypothetical future ("we might need it later" is banned).

**The ▶ Status & Next block** is the canonical resume pointer, updated as the
**last step of every task-loop commit**. It contains exactly three things a cold
Claude needs:
- **Spec:** current frozen version + one-line of the latest decision.
- **Done:** which F-tasks and S-slices are complete.
- **Next:** the exact next task, and the per-task **Loop** one-liner.

**The Carried Debt Ledger** (`## Carried debt ledger`) holds every deferral with
a stable **`DEBT-N`** handle. `DEBT-N` is the canonical reference — cite it in
commits, PRs, audits, and decisions, never a prose phrase. At slice-complete
checkpoints, the default is **fix-now over defer**; only genuinely cosmetic or
forward-bound items may carry, and the inconsequentiality call is made
explicitly, not by omission.

### Worked-examples (illustrative)

- *Demand-gating:* later milestones (M2, M3…) are listed in roadmap *order* but
  are demand-gated — candidates validated by real user/customer evidence before
  they become commitments (the numbering is labeling, not commitment), and not
  pre-built. Only the current milestone gets a task breakdown.
- *DEBT-N:* a known-but-bounded defect (say, an ID-derivation edge case proven
  non-exploitable) is logged as `DEBT-1`, scheduled for "the next slice that
  touches that code," and referenced by that handle thereafter — not fixed
  speculatively, not lost in prose.
- *Decision freeze:* a foundational choice (say, the implementation language)
  records *why* the alternative was rejected — e.g. a future requirement was
  explicitly ruled out, removing the only argument for it — so the decision
  cannot be re-opened on those grounds without first revisiting that premise.

---

# Part 2 — Adversarial Execution (The Inner Loop)

> **Operational companion.** The verbatim, spawnable agent prompts (the Section-C
> templates), the fillable `PROJECT_CONTEXT` / `AGENT_ASSIGNMENTS` scaffolding,
> and the Step-0 self-bootstrap for this loop live in
> **`adversarial-development-workflow-template.md`** (drop it into the project at
> Phase 0; its filled copy is your project's `adversarial-development-workflow.md`).
> Part 2 below is the readable summary and the rules — when you actually *run* the
> loop, spawn each agent from the template's Section-C prompts so the wording is
> exact and evolves in one place.

Every slice and every non-trivial task is built through this loop. **You are the
Orchestrator.** You do not write production code or tests yourself — you spawn
subagents and enforce process discipline. The loop's guarantee comes from
**separation of duties: no agent authors and validates the same work.**

## The Per-Task Loop (one slice)

```
context  →  TDD (RED fails first → GREEN → refactor)  →  full gate green
         →  atomic signed commit to trunk  →  update the PLAN ▶ block
```

Run inside it the adversarial step sequence below. **You may not skip, combine,
reorder, or "streamline" any step.** Fractional steps run *between* numbered
steps.

```
[0] Bootstrap PROJECT_CONTEXT (first run only)
[1] Receive task + Ambiguity Sweep (the 9-axis checklist above)
[2] Test Writer — design-doc only; writes hard-to-fake behavioral tests
[1.5] Orchestrator validates test COVERAGE + CONFORMANCE (self-run)
[3] Implementer — makes tests pass; may NOT weaken assertions
[3.5] Orchestrator's Cheap Mechanical Gate (self-run; no reviewer yet)
[4] Process Reviewer  ‖  [4.5] Domain Expert  ‖  [4.6] Security Auditor   (parallel)
[5] Iterate — zero tolerance, zero workarounds (re-run 3.5 before each re-review)
[5.5] Production Wiring Audit (fresh, uncontaminated agent)
[6] CI gate at depth-by-risk; the FINAL commit is always preceded by one full gate
```

**Model strength:** spawn every judgment-bearing agent at full strength (the
strongest model available). A downgraded reviewer/auditor is a silent hole in
the chain. A weaker model is permitted ONLY for pure pattern-matching with no
behavioral judgment: the Step 3.5 mechanical gate, or a fix round the
orchestrator has *read the diff of* and confirmed is hygiene-only (formatting/
comments/imports). When in doubt, it is not hygiene-only — use full strength.

## The PROJECT_CONTEXT Block

Every subagent receives this verbatim. It is the project's constitution in one
block. The **canonical fillable version** — `[bracket]` placeholders plus the
Step-0 self-bootstrap that populates them — lives in
`adversarial-development-workflow-template.md`; the list below is what that block
must contain. Populate during Phase 0; a human may edit it any time (e.g. after a
new anti-pattern is discovered). Sections:

- **Project** — one-paragraph what-it-is.
- **Tech Stack** — language version, key libraries, test runner, linter.
- **How to Run Tests** — every recipe (unit, integration, e2e, any specialized
  suite, single-test).
- **Cheap Mechanical Gate** — the exact Step-3.5 checks (see below).
- **Full CI Command** — the gate ladder (frequent / between-rounds / end-of-slice).
- **Production Entry Points** — the composition root(s) the wiring auditor traces from.
- **Type & Allocation Conventions** — e.g. value-object rules, precision/rounding
  policy, allocation constraints on hot paths, rich vs anemic domain.
- **Time & ID Conventions** — wall-clock/RNG policy (e.g. injected, not ambient,
  in core logic); ID/sequence generation; any version-pinning rule.
- **Concurrency Model** — e.g. event loop, thread-per-request, actor model, or
  "stateless engine + DB-as-serializer: one per-entity transaction (load →
  decide in memory → persist → commit)"; note any read-routing rule.
- **Forbidden Patterns** — the numbered list of things that are an automatic
  finding (see the example set below as a starting template).
- **Suppression Syntax** — the exact language constructs that count as sham-fix
  suppression (e.g. lint-disable pragmas, test-skip annotations, blank-assigning
  errors, build-tag exclusion, weakened assertions…).
- **Required Test Init/Teardown** — harness calls, container/fixture setup, what
  is blocking.
- **Test-discipline invariants** — "a test of class X MUST do Y" rules the
  orchestrator checks at Step 1.5 (e.g. a non-determinism-prone test must be
  isolated/gated/serial per the project's rule, or it flakes under the full
  parallel suite).

### Forbidden Patterns — an example set (template for your own)

A concrete, numbered "automatic finding" list. The items below are **illustrative**
— replace them with YOUR project's known-bug patterns. Keep the *shape*: each is a
specific, greppable anti-pattern, not a vague principle. The last two are generic
and belong on almost every list — they are the two most common AI escapes:

1. Using an imprecise type where an exact one is required (e.g. float for money).
2. An implicit/default conversion or rounding outside a defined boundary.
3. A lossy serialization on an audit/golden path (use a canonical, lossless form).
4. Ambient wall-clock or RNG inside core logic (inject them instead).
5. A module reaching past another module's public boundary (enforce with a lint).
6. Business logic in handlers/adapters/SQL (anemic domain).
7. Trusting an unauthenticated input as if it carried verified identity/claims.
8. Cross-tenant / cross-user data exposure (every scoped query carries the full
   isolation key).
9. State held across requests where the design says the component is stateless.
10. Reintroducing a banned tool/runner, or pursuing a permanent non-goal.
11. Speculative generality justified only by a hypothetical future ("might need
    it later" is banned).
12. **Phantom-in-production adapter** — a "real" value path that only
    materializes through a test-only seam, while the shipping binary gets a
    no-op / empty / zero implementation. Simulated ≠ inert: a simulated or stub
    adapter intended to ship MUST GENERATE real values on the production path.
13. **Docstring–code divergence on a load-bearing path** — a comment claiming
    behavior the code does not implement. On a load-bearing path this is a
    CORRECTNESS finding; the comment is the contract the next reader trusts.

## Dedicated Agents — The Persistent Specialist Layer

The workflow template's Section C prompts are the **per-task contract** — they
govern one spawn. A *dedicated agent definition* (e.g.
`.claude/agents/<name>.md`) is the **persistent craft layer underneath**: the
project knowledge, hard invariants, and standing judgments a role carries into
every spawn. Stock/generic agents are fine on day one; the dedicated layer is
the steady state — and it is where escape-driven convergence (Part 3)
accumulates *per role*, not just in the shared prompts.

Rules of the pattern:

- **The layered-prompt contract.** Every agent definition states it verbatim:
  the orchestrator's invocation prompt (the Section C template) governs the
  task and its rules; this file is persistent craft and judgment applied on
  top; **if they ever conflict, the invocation prompt wins.** This keeps the
  workflow's guarantees (read-only mandates, finding protocol, zero
  tolerance) in exactly one evolving place — Section C — while the agent file
  deepens, never overrides.
- **Which roles get one.** Always Step 2 (test writer) and Step 3
  (implementer) — they run every slice and accumulate the most project
  judgment. Beyond those two, derive the roster from the SPEC using the
  procedure below ("Deriving the project's specialist roster") rather than
  guessing or copying another project's crew.
- **Model pinning.** Pin the strongest model in the agent's frontmatter
  (`model: …`) — never inherit a runner default. For stock agents with no
  frontmatter to edit, the orchestrator passes the model override on every
  spawn, and `AGENT_ASSIGNMENTS` records that obligation explicitly.
- **Per-agent memory.** Each dedicated agent gets its own persistent memory
  directory (e.g. `.claude/agent-memory/<name>/`): one fact per file, plus a
  `MEMORY.md` index — one line per fact, never content, kept bounded (it is
  loaded on every spawn). The agent definition embeds the directive. Memory
  reflects what was true *when written*: the agent verifies a remembered fact
  against the repo before acting on it. The repo, not memory, is the source
  of truth.

### Deriving the project's specialist roster (bootstrap procedure)

Run this during Phase 0 Step 0.7 (and re-run it at each milestone checkpoint —
a new milestone often opens a new surface). The derivation is **two-stage:
signals trigger archetypes; the project's own stack decisions specialize
them.** This document defines only the **core archetypes** — stack-agnostic
role *shapes*. The concrete agents are deliberately not listed here, because
they do not exist until the foundational §8 decisions (language, architecture
shape, data tier, deployment target) give each triggered archetype its
specific form. The roster is derived from the SPEC, proposed to the user, and
demand-gated — never speculative.

**1. Read the signals — which archetypes does this SPEC trigger?** Walk the
frozen SPEC and note which of these the project actually commits to:

| Signal in the SPEC | Core archetype | What it owns (in any stack) |
|---|---|---|
| §1 spine invariant is intricate domain correctness (conservation, state machines, regulated rules) | **Domain-expert reviewer** | The Step 4.5 slot: invariant hypothesis testing in the domain's own terms |
| Hard latency / throughput / resource targets; hot paths | **Performance architect** | Hot-path review, measured ceilings, resource discipline |
| Non-trivial persistence: schemas, transactions, migrations, query-shaped risk | **Data/persistence specialist** | Schema and query review, transaction semantics, migration safety |
| Scale or infrastructure commitments: managed tiers, autoscaling, multi-region, cost envelopes | **Infra/scale architect** | Capacity math, deployment topology, scaling cliffs |
| §7 trust boundaries: authn/authz, tenant isolation, secrets, partner-facing surface | **Security auditor** | The Step 4.6 slot (stock usually suffices until project judgment accumulates — then dedicate it) |
| More than one deployable or module with contracts between them: microservices, a multi-module system, a frontend↔backend split | **Integration/contract specialist** | Every inter-part boundary as a drift-guarded contract; distributed failure semantics (retries, idempotency, ordering, schema evolution of events) |
| A user-facing UI surface | **Frontend/UI specialist** | Component and state architecture, accessibility, browser-runtime verification |
| A framework- or build-heavy platform: multi-module build graphs, monorepo workspaces, heavyweight frameworks | **Platform/build specialist** | Module/build-graph topology, dependency hygiene, framework idioms |
| §6 testing strategy spans multiple tiers with container-heavy or flake-prone suites | **Test architect** | Tier placement and pyramid health — deliberately distinct from the Step 2 *author* |
| A recurring, gnarly failure class (flakes, races, environment-sensitive breaks) | **Debugger** | Systematic root-causing; owns the failure-signature catalog |
| CI/CD, IaC, or release phases in the roadmap | **Deployment engineer** | Pipeline, IaC review, release discipline |

**2. Specialize each triggered archetype to the stack.** An archetype is a
role shape, not an agent. Instantiate it in the terms of the §8 foundational
decisions you just froze:

- **Name it in stack terms** — `<stack>-<archetype>`, not the abstract label.
- **Write its judgment in the stack's idioms.** What "hot path" means *here*:
  allocation-free loops in one stack, GC pressure and pool sizing in another,
  never blocking the event loop in a third. Same archetype, different craft.
- **Seed its first standing standard** from the SPEC/PROJECT_CONTEXT rules
  that archetype defends (its slice of the forbidden patterns, the relevant
  hard invariants).
- **Multiply the wiring surface, not just the roster.** A multi-deployable
  architecture means one composition root per service (plus the frontend
  bundle entry); list every one in PROJECT_CONTEXT "Production Entry Points"
  so the Step 5.5 auditor traces each deployable, and give the
  integration/contract specialist a drift guard per boundary.

Worked examples — the same archetypes, three different instantiations:

- *Single-language modular monolith over a managed SQL tier* → the
  performance archetype becomes a language-level hot-path/allocation
  architect; the infra archetype becomes a specialist on that managed tier's
  scaling cliffs; the contract archetype owns the one external API's drift
  guard.
- *Java microservices, multiple Maven/Gradle modules* → the performance
  archetype becomes a JVM specialist (GC, pools, framework lifecycle); the
  integration/contract archetype owns inter-service API/event contracts with
  a consumer-driven contract test per boundary; the platform archetype owns
  the module graph and dependency hygiene.
- *Node frontend + backend* → the frontend archetype owns components, state,
  accessibility, and browser-runtime verification; the contract archetype
  owns the frontend↔backend API as a drift-guarded contract; the platform
  archetype owns the workspace/build graph.

**3. Propose, don't create.** Present the specialized roster to the user in
the same one-round-trip confirmation as the PROJECT_CONTEXT block: each
proposed agent with one line of *why this project needs it* (cite the SPEC
section and the §8 decision that shaped it) and its first standing standard.
The user confirms, trims, or defers each.

**4. Apply the no-empty-agent rule.** An agent definition with no
project-specific judgment in it is a stock agent with a fancy name — do not
create it. Specialization gives a day-one agent real content (its craft
derives from the §8 stack decisions), but it still needs at least one
project-specific standing standard to exist; otherwise **defer it** (record
the deferral) and create it at the moment of first real signal — the first
hot-path slice, the first flake class, the first capacity question. Deferring
is the normal case for most of the table; the mandatory pair (Step 2 +
Step 3) plus one or two SPEC-obvious specialists is a typical day-one roster.

**5. Bind or pool each confirmed agent.** Two kinds of specialist, both built
from the same template:
- **Step-bound** — fills a numbered slot in `AGENT_ASSIGNMENTS` (Step 2, 3,
  4.5, 4.6) and is spawned by the workflow every round.
- **Consultation** — not bound to a step; the orchestrator spawns it ad hoc
  for design questions, checkpoint audits, capacity math, or flake
  root-causing. Record it in the agent layer anyway (definition + memory
  directory) so its judgment persists between consultations.

**6. Stand up each agent.** From the template below: definition file with
pinned model, the layered-prompt contract verbatim, its first standing
standard, and an empty memory directory with a blank `MEMORY.md` index.

### Agent-definition template (fill the brackets)

```markdown
---
name: <project>-implementer   # or behavioral-test-writer, <domain>-reviewer…
description: <Role> for <Project>'s adversarial-development-workflow
  Step <N>. <One sentence: what it does, and when the orchestrator
  spawns it.>
model: <strongest-available>  # pin explicitly — never inherit a default
---

You are the <role> for **<Project>** — <one-line project description>.
You execute **Step <N>** of the adversarial development workflow.

The orchestrator hands you the task, design doc, and step inputs in the
invocation prompt (the workflow's Section C template). That prompt governs
the task and its rules. This file is your **persistent craft, constraints,
and judgment** — apply it on top; if they ever conflict, the invocation
prompt wins.

## Your job
<The role's completion target restated in project terms — e.g., for an
implementer: "correct, production-wired code reachable from <composition
root>; every pre-written test passes for the right reasons; every reviewer
finding gets a genuine fix.">

## Project invariants you enforce
<The PROJECT_CONTEXT rules this role most often defends, in this role's
terms: hard invariants, forbidden patterns, required idioms.>

## Standing standards
<Operator-ratified judgments this role owns that outlive any one task —
e.g., a test architect's tier-placement rules. Grows via Part 3's
evolve-from-escapes loop.>

## Memory
You have a persistent file-based memory at
`.claude/agent-memory/<name>/`. One fact per file; `MEMORY.md` is the
index — one line per fact, never content, keep it bounded. Before acting
on a remembered fact, verify it against the repo: memory reflects what
was true when written; the repo is the source of truth.
```

## The Steps in Detail

**Step 0 — Bootstrap PROJECT_CONTEXT.** First run only; otherwise skip.

**Step 1 — Receive task + Ambiguity Sweep.** Parse the task; gather the design
doc / acceptance criterion / linked issue. If vague, STOP and ask. Run the 9-axis
sweep (Part 1) and raise all ambiguities at once.

**Step 2 — Write behavioral tests (Test Writer agent).** The test writer works
from the **design document only** — never sees implementation, never writes
implementation. Tests must be **hard to fake**:
- **Externally observable** — assert on an HTTP response, counter, emitted
  message, persisted row. "Field is non-null" is not proof.
- **Specific to the new path** — if the old code could pass it, it is worthless.
- **At least one negative test** — disable/bypass the new component and assert
  the operation FAILS. Strongest defense against phantom wiring.
- **Pipeline-complete** — for every data source and deliverable, a test proving
  data flows source → new code → observable output, through the same entry
  point a real user uses. Component-API tests do not satisfy this. *"Test the
  pipeline, not just the API."*
- **Boundary tests** — zero, max, one-over, one-under, out-of-order, mid-op failure.
- Per test, state its **anti-fake property**: why it cannot pass via the old
  path or phantom wiring.

Return the test writer's output **verbatim** to the next steps — never
summarize/filter.

**Step 1.5 — Validate coverage + conformance (orchestrator, self-run).** After
Step 2, before Step 3:
- *Coverage (1–4):* list every data flow and data source in the design doc; for
  each, confirm at least one test proves data enters from that source and
  reaches an observable output. Zero tests for a flow → re-spawn Step 2.
- *Conformance (5):* check every test against the PROJECT_CONTEXT
  test-discipline invariants (build-tag gating, required helpers, parallelism
  constraints). A coverage-complete test that violates a discipline invariant is
  a Step-1.5 FAIL — fix the scaffold before Step 3.
- *Cross-slice arithmetic (6):* for every test SETUP quantity that feeds a
  **prior-slice frozen formula** (a threshold/rate/level already shipped),
  verify the arithmetic matches the test's stated premise. A setup that silently
  crosses a boundary the test assumes never fires is a FAIL — catch it here, not
  at review.

**Step 3 — Implement (Implementer agent).** Makes the pre-written tests pass.
**May NOT weaken assertions** (change expected values, remove assertions, add
skips). MAY change test setup/mocks/helpers. If test infra conflicts with good
production code, fix the *infra*, never degrade production. Every component must
be **stored as a field, instantiated in production code, and called from a
production path.** No stubs/TODOs on runtime paths. On fix rounds, every finding
gets a **genuine** code change with a file:line before→after, and a per-finding
rationale; no "pre-existing" dismissals, no partial cleanups. Deliverable: the
code, a **data-flow trace** (production entrypoint → component → observable
effect), pasted test output, and an honest "what is NOT done."

**Step 3.5 — Cheap Mechanical Gate (orchestrator, self-run, EVERY round).** A
fast deterministic pre-filter so reviewers spend attention on judgment, not on
catching a build break or a fake green. Run all of (adapt to language):
1. Formatter check is clean.
2. Production build (no test tags) compiles.
3. Vet/static-analysis clean.
4. Lint: zero issues.
5. **Fabricated-green guard** — independently re-run the exact tests the
   implementer claimed green, yourself, from a clean tree. **If RED, return to
   Step 3 — do NOT spawn reviewers.** A green you did not reproduce is not a green.
6. **Phantom-adapter grep** — confirm the production composition root
   instantiates the concrete adapter *unconditionally* (not behind a test-only
   seam) and its primary method returns real values on the non-test path.

This does NOT replace the reviewers or the wiring audit.

**Steps 4 / 4.5 / 4.6 — Review (parallel, independent).** Spawn in a **single
message** so they run concurrently and **cannot see each other's output** —
non-anchoring is the entire point. All are **strictly read-only**: they analyze
and report; they NEVER edit/patch/mutate the tree, not even transiently. If a
reviewer suspects a test is vacuous, it REPORTS the finding with the exact
mutation described (file:line before→after) — the *orchestrator* runs any
mutation-proof.
- **Step 4 — Process Reviewer.** Data-flow verification (is each component
  stored/instantiated/called from production; would removing it break a test?),
  plan-coverage, test verification (does it exercise the new path; would it pass
  with the new code deleted?), completeness (stubs, suppressions, forbidden
  patterns), phantom-in-production and docstring-fidelity checks.
- **Step 4.5 — Domain Expert.** Invariant hypothesis testing: for every state
  variable changed, trace bounds, error paths, cross-module readers, temporal
  ordering — conclude SAFE or BUG per hypothesis with evidence. Plus
  domain-specific axes (allocation/locks if hot-path; determinism if replayable;
  transactional correctness if persistence-touching). Tag each finding
  CONFIRMED / LIKELY / SUSPECTED.
- **Step 4.6 — Security Auditor.** **Mandatory** for any change touching
  auth/authz, multi-tenant isolation, input parsing, secrets, crypto, error/log
  output, or dependencies; cheap and trust-calibrating otherwise. Threat-model
  first (attack surface, principals, assets, abuse cases), then line review:
  BOLA/IDOR, privilege escalation, fail-open paths, injection, secret
  disclosure, DoS on the changed path, supply chain, determinism-as-security.
  Every finding states a concrete exploit and a traced reachability chain.

Merge findings into the tracker with `FIND-<round>-<n>` IDs **+ reviewer name**.

**Step 5 — Iterate (zero tolerance, zero workarounds).** If any reviewer finds
any issue at any severity:
1. Every finding enters the tracker as `OPEN`.
2. Re-spawn the implementer with the **full** finding list.
3. Re-run Step 3.5 (the fabricated-green guard re-runs every round).
4. Re-spawn **all** reviewers with updated code + the prior tracker. They
   **verify prior findings FIRST**, classifying each `GENUINELY_FIXED |
   SHAM_FIX | NOT_ADDRESSED | PARTIALLY_FIXED | REBUTTED`, then look at new code.
5. Any `SHAM_FIX` → send back with explicit "this was a sham fix" instruction;
   trust is reset. A sham fix is **worse** than the original finding.
6. Repeat until **all reviewers report zero findings on the SAME round AND every
   prior finding is GENUINELY_FIXED or REBUTTED-with-reviewer-agreement.**

*Delta-scoped new-code review is allowed on bounded fix rounds* (search for
*new* problems in the changed hunks + one call-hop out). **Prior-finding
verification is NEVER scoped down** — every open finding is re-verified at full
rigor every round.

**Step 5.5 — Production Wiring Audit (fresh agent, mandatory).** A new agent
that has *not* seen prior rounds (uncontaminated). It traces every new component
from a production entry point and classifies it `PRODUCTION_WIRED |
PHANTOM_WIRED | STUB_ONLY`. This exists because the most common AI failure mode
is code that compiles, passes unit tests, is exported from the module root, and
is **never called from production.** Any `PHANTOM_WIRED`/`STUB_ONLY` → back to
the implementer. Also read-only (reason about the removal test, describe the
deletion; the orchestrator runs any destructive proof).

**Step 6 — CI gate + report.** Run CI at depth matched to change risk:
behavior-change → the full end-of-slice gate; hygiene-only iteration → the cheap
checks + targeted tests. **The FINAL commit is ALWAYS preceded by one full
gate**, regardless of how intermediate iterations were gated. Report the facts:
files changed, each reviewer's clean assessment, the finding-tracker summary
(rounds, total findings, final status each), and the verify command. Do not
editorialize about thoroughness.

## The Finding Tracker

Maintain from the first review round to completion:

```
ID (FIND-<round>-<n>) | Round | Reviewer | Severity | Location | Status
```

`Status: OPEN → GENUINELY_FIXED | SHAM_FIX | NOT_ADDRESSED | PARTIALLY_FIXED | REBUTTED`

- Pass the tracker into every re-review round. **Do not lose findings.**
- The implementer may **not** unilaterally close a finding — only a reviewer on
  the next round marks `GENUINELY_FIXED`.
- A `REBUTTED` finding requires the implementer's evidence-based case (code
  trace / test output) that a reviewer re-evaluates. Silent dismissal forbidden.

**Severity taxonomy** — every label means "must fix before proceeding," even
`NOTE`:
- **BLOCKER / WARNING / NOTE** — all must fix (yes, NOTEs count).
- **SHAM_FIX** — a prior finding "fixed" by suppression/workaround/relocation/
  weakening. Worse than the original; triggers mandatory re-work and resets trust.

**Zero-tolerance exit:** ALL reviewers report zero findings at ANY severity on
the SAME round. Not "almost clean," not "only NOTEs left." Zero.

## The Gate Ladder (Green From Every Tier)

Define three tiers and use them at the right moments. A unit-only green is a
**dev crutch, not a commit gate** — final commit requires green from *every*
tier (unit + integration + e2e + any specialized/replay suite).

| Tier | When | What |
|---|---|---|
| **frequent** (fast gate) | session baseline; between rounds | Build + unit/fast tests, no containers. Seconds. Cannot hit the container-settle/flake class. NOT sufficient for commit. |
| **between-rounds** (integration gate) | when a round needs integration coverage | format + static-analysis + lint + vuln + full test (incl. integration/e2e). |
| **end-of-slice** (full gate) | **pre-commit ONLY** | the integration gate + any specialized/replay suite. **Bounded-generous** timeouts (above the worst legitimate runtime, but bounded so a hang fails fast instead of churning for hours and leaking containers). Stream per-test output — never run a long serial gate blind. |

Discipline:
- **Pin tool versions so local ≡ CI.** One task runner; never reintroduce a
  banned one.
- **Host-load container timeouts are environmental flakes, not logic failures —
  but never hand-wave them.** Root-cause each by **isolated re-run** (run that one
  test alone, serially, with no parallelism — it passes in seconds). A real defect
  is a reproducible assertion/output diff *in isolation*; a parallel-starvation
  flake passes isolated. **Zero flaky tests** is the standard — flakes are
  blockers, root-caused by construction, never suppressed.
- **Never run two container-heavy suites concurrently** — overlapping boots
  starve the settle gate. Serial is the most flake-resistant.
- "Green" is defined ONLY by the prescribed recipes — never an ad-hoc
  over-parallelized `test ./...`, which manufactures settle flakes.

## Commit Discipline

- **Trunk-based.** Commit small, atomic, **signed** changes straight to the main
  branch (no long-lived feature branches). Incomplete user-facing work is
  feature-flagged on trunk, not parked on a branch.
- **Each commit leaves the full gate green** — run it yourself, wait for green,
  *then* commit. Never commit in parallel with the gate; never trust a
  subagent's report of green.
- **Commit isolated units.** If the tooling bundles all modified files (acts
  like `-a`), commit explicit paths and verify with a stat view.
- **Update the ▶ Status & Next block in the same commit.** This is the last step
  of every task loop — it is what makes the next cold-start resume cheap.
- A commit message traces to the SPEC AC / `Dxx` / PLAN task it implements, and
  references any `DEBT-N` it touches by handle.

This section assumes a single human/orchestrator pair — one ▶ pointer, one
gate-then-commit pipeline. Running more than one pair? The team adaptation —
lanes partitioned on the context boundary, a spec steward, a merge queue with
a rebased-gate rule — is documented in **`TEAMS.md`**. The inner loop is
untouched by it.

---

# Part 3 — Checkpoints, Evolution, and Institutional Memory

## Checkpoint Audits

At named milestones — **Foundation complete**, **first risk-gated/critical-path
slice**, and **each slice-complete** — do not just proceed. Run an **independent
audit**: spawn fresh reviewer agents (a test-strategy auditor + a code-quality
auditor) that did not build the work, and obtain **human sign-off** before
continuing. The checkpoint is a gate, not a ceremony — it has caught
cross-slice arithmetic drift that per-slice review missed.

At a slice-complete checkpoint the default is **fix-now over defer.** Only
cosmetic or genuinely forward-bound items may become `DEBT-N`; surface the
inconsequentiality judgment explicitly in the gate questions, never by silence.

## Evolving the Workflow (Convergence)

**Every bug that escapes all agents reveals a missing rule.** When something
slips through:
1. Add the missing check as a new rule in whichever agent *should* have caught it.
2. Add the anti-pattern to PROJECT_CONTEXT's Forbidden Patterns (numbered,
   greppable).
3. Record the escape and the rule it produced (a short "escapes folded into vN"
   note), so the *why* survives.

This is what makes the workflow **converge** rather than re-litigate the same
class of bug. The phantom-in-production and docstring-divergence forbidden
patterns (items 12/13 above) and the Step-1.5 conformance checks were all born
this way — each from a specific escape.

### The meta-rule for any "make it faster/cheaper" proposal

Before adopting any efficiency change, classify it. **ACCEPTABLE** if it only
changes:
- **WHEN** a check runs (a cheap mechanical gate before reviewers);
- **HOW MUCH** surface is re-read (delta-scoped new-code review);
- **WHAT MODEL** runs a pure pattern-matching step (model tiering);
- **HOW DEEP** verification goes, scaled to change-risk (depth-by-risk CI).

**REJECTED — it is erosion, not efficiency** — if it changes:
- **WHO validates** (never an author of the same work — separation of duties);
- **WHETHER the orchestrator independently verifies** (always — never trust a
  summary; re-run, re-read, re-confirm yourself);
- **WHETHER NOTEs can be waived** (they cannot — zero-tolerance is absolute).

Classify by *effect on those two lists*, not by how the change is described.
"It only saves a round" is how erosion is always sold.

## Institutional Memory (Persistent Across Sessions)

Four durable records keep a cold-start Claude from re-deriving settled ground:
1. **The §8 decision log** — *why* every design choice is what it is (+ rejected
   alternatives). Skim it on resume; never re-derive a frozen decision.
2. **The DEBT-N ledger** — what was deliberately deferred and when it is due.
3. **A file-based memory** (if your environment provides one) — user
   preferences, working agreements, and hard-won "do it this way" lessons, one
   fact per file, linked. Memory is background context that reflects what was
   true *when written* — if it names a file/flag/function, **verify it still
   exists** before acting on it. The repo, not memory, is the source of truth.
4. **Per-agent memory** (Part 2 § "Dedicated Agents") — each dedicated agent's
   own memory directory, holding the role-scoped lessons a specialist would
   otherwise re-derive expensively (a measured performance ceiling, an
   operational invariant that isn't in the spec, a flake class root-caused
   once). Same caveat as 3: verify against the repo before acting on it.

## Is It Working? — Process Health Signals

The process converges or it erodes; you do not get to assume which. These are the
signals that say it is converging: **rounds-per-slice trending down** for slices of
comparable complexity; **escapes-per-slice trending toward zero**; **zero flaky
tests** holding (not creeping back as suppressions); the **forbidden-patterns list
growing early then plateauing** (each escape adds one, then the class stops
recurring); and **checkpoint audits finding less each time**.

The data source is free. The commit-message convention already encodes the slice
and round (`Sx Rx`) and the decisions (`Dxx`), so `git log --oneline` **is** the
process dataset — one grep recovers rounds-per-slice, another the decision count.
You do not instrument anything; the discipline that makes commits traceable makes
them measurable.

Calibration anchors from the reference project: a per-slice round count with a
**median of ~3–4 and a maximum of 12** (the hardest slice), and **3 escapes across
~18 slices** — each one now a permanent rule. Use those as a yardstick, not a
target: a project whose numbers are wildly lower is more suspect than one whose
numbers match.

The warning sign that is **not** health: rounds dropping because the reviews got
*softer*. Convergence and erosion both reduce round count, and they look identical
on a graph. Distinguish them by mechanism — convergence means fewer findings
because **earlier gates catch them or rules prevent them**; erosion means fewer
findings because **rigor decayed**. So when rounds drop, ask *which gate is catching
things now*: if escapes moved to Round 1 because a forbidden pattern or the Step 3.5
grep now flags them, that is convergence. If the answer is "none — they just stopped
finding things," re-read the meta-rule (§ "The meta-rule for any 'make it
faster/cheaper' proposal"). A drop you cannot attribute to a specific gate is the
erosion the whole process exists to prevent.

## Anti-Patterns to Watch For (the orchestrator's own failure modes)

- **Orchestrator cheating** — weak reviewer prompt, summarized output hiding
  problems, skipping a reviewer. *You are the weakest link; your completion bias
  is the threat.*
- **Combining steps "for efficiency"** — one agent writes tests and code; skip
  the wiring audit "because reviews were clean." That is completion bias talking.
- **Stopping short of zero** — "mostly clean," "only NOTEs remaining." A NOTE is
  a finding.
- **Turning a reviewer into an author** — injecting a "mutate then revert"
  instruction into a read-only reviewer prompt. Reviewers report mutations; the
  orchestrator runs them.
- **Fabricated green** — handing reviewers a green claim you never reproduced.
  Re-run it yourself (Step 3.5).
- **Pre-existing dismissal** — "that existed before my change." No carve-out: if
  a reviewer found it in code the task touched, it is in scope.
- **Trust-reset after parallel mutation** — when reviewers/implementers have
  touched a shared tree, the orchestrator independently re-verifies tree
  integrity and runs the final gate *itself* before commit.

---

## When NOT to Use the Full Inner Loop

- Exploratory prototyping (the goal is to discover requirements, not satisfy them).
- Throwaway scripts / one-off migrations.
- Pure refactors with full existing test coverage (the existing tests are the adversary).
- Changes too small to justify it (typos, renames, single-line fixes).

For small-but-non-trivial changes, run the minimal trio — **Test Writer →
Implementer → Process Reviewer** — and still run the **wiring audit** (cheap;
catches the most common AI failure mode). Apply the full loop wherever the cost
of a production bug exceeds the cost of the process.

---

## The `/next` Command — One-Command Steady State

Encode the entire steady-state loop as a **single repeatable slash command** so
a session can be resumed and advanced with one trigger. The command does three
things in order: (1) establish the trunk-green baseline via the Resume Protocol,
(2) continue the Next task via the per-task loop, (3) drive that task through the
adversarial inner loop as Orchestrator. Create it once during Phase 0 (e.g.
`.claude/commands/next.md`); thereafter "do the next thing correctly" is one word.

A project-agnostic template (distilled from a working version of this command):

```markdown
---
description: Resume work — run the resume protocol, then execute the next task
             via the adversarial development workflow
category: project-task-management
---

Resume work exactly where the last session left off, then drive the next task
through the adversarial development workflow. The repo — not conversation
memory — is the source of truth.

## 1. Establish the trunk-green baseline (resume protocol)
Read, in order, before acting:
1. PLAN ▶ Status & Next block — spec version, Done, the exact Next task, the loop.
2. SPEC status header, then skim §8 (the dated Dxx ADR log) for the *why*.
   Frozen decisions are settled — do NOT re-derive them.
3. Run `git log --oneline -15` and the **fast baseline gate** (the Docker-free
   build + unit/app tier — the prior slice's end-of-slice gate already
   established integration/e2e green, so a pure resume needs only this). Confirm
   trunk-green matches "Done". If it is not green, STOP and report — never start
   new work on a red baseline. (A project whose commit-loop trigger doubles as
   its resume command may run the heavier end-of-slice gate here instead; pick
   one and keep the Resume Protocol's gate identical to it.)

## 2. Continue the Next task via the per-task loop
context → TDD (RED first → GREEN → refactor) → full gate green → atomic signed
commit to trunk → update the PLAN ▶ block in the SAME commit.
Rules that bite:
- Undefined behavior → STOP and raise a §8 decision (versioned revision + dated
  Dxx row). Never a silent choice.
- Spec changes are versioned revisions, never silent mid-build patches.
- Every change traces to a SPEC AC / Dxx / PLAN task. No scope creep.
- At checkpoints (Foundation / risk-gated / slice-complete): run the independent
  reviewer-agent audit and get human review before proceeding.

## 3. Execute the task under the adversarial workflow
You are the Orchestrator. Follow it exactly — every numbered and fractional
step, in order, no skipping/combining/reordering:
@adversarial-development-workflow.md   (the project's filled-in copy of
adversarial-development-workflow-template.md; or Part 2 of this playbook)

Only report success when the zero-tolerance exit condition is met: all reviewers
report zero findings on the same round, every prior finding is GENUINELY_FIXED,
the wiring audit is clean, and the full gate is green.
```

The discipline this buys: the loop is **never** improvised from memory. Every
resume runs the same baseline check, every task runs the same adversarial
sequence, and the ▶ block is updated every time — so the *next* `/next` starts
clean. One command makes the correct path also the easiest path.

## Resume Protocol (Fresh Instance: Do This FIRST)

Conversation memory is **not** the source of truth — the repo is. To pick up
exactly where the last session left off (this is what `/next` step 1 automates):

1. Read the auto-loaded project instructions (`CLAUDE.md`/`AGENTS.md`).
2. **PLAN → the ▶ Status & Next block** at the very top: current spec version,
   what's Done, the exact Next task, and the loop. This is the canonical resume
   pointer.
3. SPEC status header + skim §8 (the Dxx ADR log) for the *why*. **Do not
   re-derive frozen decisions.**
4. `git log --oneline -15` and run the **frequent gate** — confirm the baseline
   is clean. (The prior slice's end-of-slice gate already established the
   integration/e2e/specialized suites green; don't re-run the full heavy gate
   just to resume.)
5. Continue the Next task via the per-task loop: context → adversarial inner
   loop → full gate green → atomic signed commit to trunk → update the ▶ block,
   in the same commit.

**Rules that bite, restated:** undefined behavior → STOP and raise a §8 decision
(versioned revision + dated Dxx row), never a silent choice. Spec changes are
versioned revisions, never silent patches. The process is the product —
execute it exactly.
