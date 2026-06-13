# Changelog — Template Versions

This is the version history of the **template itself**, not a git log of the repo.
Entries are fed by the fold-back contract (README "Background" + Section G): a
generalizable escape folded into a user's local copy is contributed upstream and
becomes the next template version, recorded here.

## v2 — 2026-06 (current)

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

## v1

Original release — the core adversarial inner loop: a blind test writer (writes
behavioral tests before any code, never sees the implementation), a non-weakening
implementer (must pass the tests, forbidden from softening assertions), parallel
independent reviewers (never see each other's output), a fresh wiring auditor
(confirms production reachability), the finding tracker, and the zero-tolerance
exit (all reviewers, zero findings, same round).
