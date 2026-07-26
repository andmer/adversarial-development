# Changelog — Template Versions

This is the version history of the **template itself**, not a git log of the repo.
Entries are fed by the fold-back contract (README "Background" + Section G): a
generalizable escape folded into a user's local copy is contributed upstream and
becomes the next template version, recorded here.

## v3 — 2026-07 (current)

v3 addresses the first defect *class* the kit was structurally blind to, rather
than hardening a check it already had. **The absence class: nothing written is
wrong; something unwritten is missing.**

Every technique in v1/v2 compares one artifact against another derived from the
same understanding — tests↔code, coverage↔lines, mutation↔code, types↔uses,
wiring↔components, review↔diff. Each can only find *contradictions inside* the
system, and an absence is not a contradiction. In formal terms the suite
accretes **safety** properties ("nothing bad happens" — witnessed by a finite
trace, cheap, always terminates) and drifts structurally blind to **liveness**
("something good eventually happens" — untestable unless someone names the good
thing). The SPEC layout showed it plainly: §7 Boundaries and §11 Permanent
non-goals are both "what NEVER happens," and there was no section for what must
keep happening.

- **SPEC §14 — the standing-obligations register** (`OB-n`). The liveness twin
  of §7/§11: what users are owed and must keep being owed, with greppable keys
  (including data-shape triggers) and a proof per row. Mutation requires a
  versioned revision plus a `Dxx`. **Authored by the operator, never by an
  agent** — it is the only input to the process not derived from the process;
  an agent writing it would close the loop it exists to break. Generalizes
  `FRZ-N` from frozen arithmetic to live obligations, and brings it to
  single-operator scale.
- **The Proof contract — execution through the front door.** An obligation is
  only proven by a check that enters a declared **Production Entry Point** and
  asserts on user-observable output. Every other check in the kit analyzes
  artifacts; execution is the one method that fails *differently* — and it is
  what a human QA is doing when they catch this class in a single session. A
  proof that constructs its scenario mid-system is `PROOF-INTERIOR` and counts
  as unproven. PROJECT_CONTEXT's entry points are now what checks are *run
  from*, not only *traced from*.
- **Step 1.5 check 7** — the threat-scoped sweep. `KEYLESS` rows fail closed to
  forced judgment, and `KEYLESS` is the *common* case here: a liveness property
  usually has no symbol to grep, which is exactly why it escapes.
- **Step 5.5 Standing-Obligation Re-Verification** — the wiring audit run
  backwards, in the same spawn: not "is the new code reachable?" but "is the old
  obligation still reachable?" `STILL_FIRES | BROKEN | UNPROVEN`. **It derives
  its own scope** (every 1.5 hit, plus every `KEYLESS` row unconditionally, plus
  every key hit against the real diff) — inheriting 1.5's list would give two
  gates one blind spot, which is one gate.
- **Step 6 totality gate** — completeness measured, not claimed. It explicitly
  does **not** prove correctness: a green gate is compatible with a broken
  obligation.
- **Full re-verification at each slice-complete checkpoint** — the latency
  bound. Per-slice checks are threat-scoped, so a single missed sweep hides an
  obligation forever. Re-executing every proof at each checkpoint converts
  worst-case survival from *unbounded* to *one slice*.
- **OPERATOR Moment 6** — the five moments become six. The first five check the
  agents' work; the sixth supplies information the agents cannot obtain.
- **Method diversity (Section D, Section G rule 4)** — separation of duties buys
  independent *judgment*, not independent *method*. **Agreement among
  same-method checks is not evidence.**
- **Derivation over assertion (Section G rule 5)** — names the generative
  principle behind the phantom grep, greppable forbidden patterns, `FRZ-N`, and
  §14 itself: convert assertions into derivations, or into checks.
- **§8: a decision that redefines a term must enumerate the artifacts citing
  it** — the anti-rot rule for every catalog in the kit.

No step was added to the sequence and no seat was added to the roster: Steps 1.5
and 6 are orchestrator-run, and the Step 5.5 mirror rides the spawn that already
exists. **Zero new agent spawns per slice.**

Escapes that drove v3: a **resting order that silently stopped firing** when two
independently-correct slices composed badly — conservation, ledger balance,
determinism, and isolation all held, and it ran undetected for two months until
found by accident (→ §14, check 7, the 5.5 mirror, the checkpoint
re-verification); **three reviewers plus a purpose-built AST tool independently
confirming a false claim**, all four classifying SQL by statement keyword
against a keyword-free fragment (→ the method-diversity rules); and a **`Dxx`
that redefined "terminal"**, silently inverting a catalog entry citing the old
meaning (→ the §8 enumeration rule).

## v2 — 2026-06

v2 hardened v1's core loop with efficiency gates that change only *when/how-much/
which-model/how-deep*, never *who validates*. Over v1 it added:

- **Step 1 Ambiguity Sweep** — the 9-axis pre-slice checklist; raise every open
  load-bearing axis at once, so each is one fewer implement→review→fix loop later.
- **Step 3.5 Cheap Mechanical Gate** — orchestrator-run pre-filter before any
  reviewer, including the **fabricated-green guard** (independently re-run the
  implementer's claimed-green tests) and the **phantom-adapter grep**.
- **Step 1.5 conformance checks 5–6** — test-discipline-invariant conformance and
  cross-slice arithmetic consistency, on top of the original coverage checks.
- **Step 4.6 Security Auditor** — a first-class parallel reviewer with its own
  threat-model-first prompt, mandatory for security-relevant changes.
- **Strict read-only reviewer mandates** — reviewers/auditors never mutate the
  tree; they report the mutation and the orchestrator runs the proof.
- **Delta-scoped fix-round review** — new-issue hunt may scope to changed hunks;
  prior-finding verification never scopes down.
- **Model tiering** — strongest model for every judgment-bearing spawn; a smaller
  model only for pure pattern-matching steps.
- **Depth-by-risk CI** — gate depth scaled to change risk; the final commit always
  gets one full gate.
- **The efficiency-vs-erosion meta-rule** (Section G) — classify any optimization
  by effect, not description.

Escapes that drove v2: a **phantom-in-production adapter** (green in tests, inert
in the shipping binary → Forbidden Pattern 12, the Step 3.5 grep, reviewer-prompt
blocks); **coverage-complete-but-nonconformant pre-written tests** (→ Step 1.5
checks 5–6); and a **reviewer that mutated the tree to prove a point** (→ the
strict read-only mandates). Each is documented in `worked-example.md` and in
`adversarial-development-workflow-template.md` Section G.

### 2026-06-12 — outer-loop additions

- **Phase 0.7** — the `CLAUDE.md` constitution plus the agent crew stood up at
  bootstrap.
- **The Dedicated Agents section** — the persistent specialist layer, with
  archetype-based roster derivation from the frozen SPEC.
- **`OPERATOR.md`** — the human operator's manual (the five moments, the refusals,
  week-one calibration).
- **`worked-example.md`** — an annotated real slice through the loop.
- **The work-decomposition vocabulary** — `Mx` / `Fx` / `Sx` / `Dxx` / `ACn` /
  `Rx` / `FIND-<round>-<n>` / `DEBT-N` as one shared identifier set.
- **The escape fold-back contract** — local fold-in plus upstream contribution.
- **Process health signals** — convergence-vs-erosion diagnostics from the
  `git log` dataset.
- **`TEAMS.md`** — the team adaptation: lanes partitioned on the lint-enforced
  context boundary, the spec-steward role, the merge queue with the
  rebased-gate rule, the **frozen-since register** (`FRZ-N` — the mechanical
  form of the cross-lane arithmetic check), and the operator ceiling.

### 2026-07-08 — Step 0.5: project agent-roster generation

- **Step 0.5 (Derive & Generate the Project's Agent Roster)** — a first-run
  step in the executable template, between Step 0 and Step 1. It folds the
  `build-methodology.md` roster-derivation procedure into the inner-loop playbook
  so an orchestrator running the template standalone actually generates **this
  project's tech- and domain-specialized agents** — the mandatory Step 2/3 pair
  plus the SPEC-obvious specialists — instead of falling back to stock
  `general-purpose`. Deriving is mandatory; creation stays demand-gated
  (no-empty-agent rule, propose-don't-create). `AGENT_ASSIGNMENTS` now documents
  the stock names as a pre-bootstrap fallback that Step 0.5 overwrites, and the
  Overview diagram, the "Read This First" responsibilities, and the step sequence
  (`0 → 0.5 → 1 → …`) are updated to match.

## v1

Original release — the core adversarial inner loop: a blind test writer (writes
behavioral tests before any code, never sees the implementation), a non-weakening
implementer (must pass the tests, forbidden from softening assertions), parallel
independent reviewers (never see each other's output), a fresh wiring auditor
(confirms production reachability), the finding tracker, and the zero-tolerance
exit (all reviewers, zero findings, same round).
