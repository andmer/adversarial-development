# Worked Example — One Slice Through the Loop

This is a real slice from the author's production project — a determinism-critical
financial simulation engine — lightly anonymized and reconstructed from its commit
history and process records. Names of domain symbols have been changed to neutral
ones; the counts, severities, round structure, decision IDs, and the escape are
true to what the history records. It is presented so you can **calibrate what
normal looks like**: this slice took **10 review rounds** and surfaced findings in
six of them, and that is the process *working*, not failing. Where the record
preserved exact findings they are quoted; where it preserved only counts and
classes, the finding rows are a **representative reconstruction** consistent with
the commits, and are marked as such.

The slice is **S8d** — overnight financing (a per-position "rollover" charge)
applied to open positions when the engine clock crosses a daily boundary. It is
the slice the kit's phantom-in-production rule is named after.

---

## The setup

**The task (one paragraph).** Every open position carries an overnight financing
charge when the simulated clock crosses the 17:00 day boundary: a signed amount
posted to the account-holder's ledger and an equal-and-opposite amount to the
house account, derived from a per-instrument rate, the position's notional, and the
number of days held. It had to be **deterministic** (byte-identical across
re-runs and process restarts — the engine has a replay/audit contract), conserve
value (the system-wide ledger sum must stay zero), and fire from the existing
always-on evaluation pump rather than a new scheduler.

**Pre-ratification (the outer loop, before any test was written).** S8d opened
with a **pre-slice decision**, `D70`, that ratified **six** load-bearing degrees
of freedom in a single questionnaire to the operator — the outer-loop equivalent
of the Step-1 ambiguity sweep, run before spawning the test writer:

1. **Sign convention** — one carry-aware *signed* rate per (instrument, value
   date); long↔short flips symmetrically; charge = signed-rate × notional × days.
2. **Counterparty / conservation** — the house account is the equal-and-opposite
   side (no new account class); holder debit ↔ house credit keeps the ledger sum
   unchanged *by construction*.
3. **The 3×-boundary rule** (a triple charge on one weekday) — **embedded in the
   rate stream**, pre-multiplied by the adapter; the engine gets **no calendar
   branch**.
4. **Event grammar** — the exact pipe-delimited wire shape of the emitted charge
   record, field names and ordering pinned.
5. **Cadence** — a **per-account boundary derive**: read the account's last-charged
   value-date under a row lock, fire once if it is behind the current boundary,
   advance it in the same transaction. The engine holds **no in-memory state**
   about rollover (restart-idempotent by construction).
6. **Held-position definition** — a position counts if it was open *at the
   boundary tick instant*, derived deterministically from the canonical sequence,
   not a fuzzy time window.

The design **pinned** all six, plus the decimal-precision policy (full precision
in the multiply, banker's rounding only at the ledger-write boundary) and the
provenance of the rate stream (a *simulated deterministic generator* behind a
port, engine-version-tagged — the real-vendor adapter explicitly **deferred** to
a later milestone). What it deferred: nothing load-bearing was left open — which
is exactly why almost all of the 10 rounds were about *implementation* defects,
not *spec* ambiguity. (Contrast: where an earlier slice left an axis open, it cost
a full implement→review→fix re-loop. D70 is the win the kit's ambiguity sweep was
generalized from.)

---

## The rounds

The orchestrator ran the standard sequence per round:
**Step 2** (test writer) → **Step 1.5** (coverage + conformance) → **Step 3**
(implement) → **Step 3.5** (cheap mechanical gate) → **Steps 4 / 4.5 / 4.6**
(process / domain / security reviewers, parallel) → **Step 5** (iterate) →
**Step 5.5** (wiring audit) → **Step 6** (CI). Reviewers were `code-reviewer`
(process), a backend/domain architect (Step 4.5), and `security-auditor` (Step
4.6). Ten rounds reached the zero-tolerance exit. A representative slice of the
trail:

### Step 2 → Step 1.5 — tests first, coverage checked before any code

The test writer produced the acceptance suite from the design doc alone: a
multi-position E2E charge test, a conservation test (ledger sum stays zero across
the charge), a cross-account isolation test (charging account A leaves account B
untouched), boundary tests for the held-at-instant definition, and a **negative
test** (a replay path that must reject an unknown event type). Step 1.5 confirmed
each data source reached an observable output. This is where the *test writer
never sees the implementation* — so it cannot quietly write tests the eventual
code happens to pass.

### Round 1 — four BLOCKERs (this is normal for a non-trivial slice)

| ID | Reviewer | Severity | Issue (one line) | Status |
|----|----------|----------|------------------|--------|
| FIND-1-1 | process | BLOCKER | A multi-position acceptance test reported green that the orchestrator's Step 3.5 re-run did **not** reproduce — a fabricated green | GENUINELY_FIXED (R2) |
| FIND-1-2 | process | BLOCKER | A `FillInputs` error path swallowed instead of failing closed | GENUINELY_FIXED (R2) |
| FIND-1-3 | domain | BLOCKER | The emitted charge record violated the pinned recorder/event-grammar contract | GENUINELY_FIXED (R2) |
| FIND-1-4 | process | BLOCKER | The rate-lookup install used a `LIKE`-pattern match that could bind the wrong instrument | GENUINELY_FIXED (R2) |

The fabricated-green (FIND-1-1) is the one to internalize. The implementer
*reported* the suite passing; the **Step 3.5 cheap mechanical gate** re-ran the
exact command from a clean tree and it was **red**. Under the rules, no reviewer
is even spawned on a red re-run — but here the gate's job was to make the claim
checkable. **A green you did not reproduce is not a green.**

### Round 2 — prior findings verified first, then new ones; two spec ambiguities escalated

R2 reviewers verified all four R1 findings as `GENUINELY_FIXED` (one, a clamp on
an impossible-under-invariant path, was `REBUTTED` with a verified code trace —
the implementer made the evidence-based case and a reviewer accepted it; silent
dismissal would have been forbidden). They then opened eight new findings (deduped
to seven unique roots). Two of them were not code defects but **spec ambiguities**
the convergent reviewers surfaced — which the outer loop absorbed as a new dated
decision, **D76**, rather than letting the implementer guess:

| ID | Reviewer | Severity | Issue (one line) | Status |
|----|----------|----------|------------------|--------|
| FIND-2-1 | process | BLOCKER | The negative replay test had two `TODO` markers blocking the load-bearing `Replay()` call and referenced a non-existent error symbol — a **vacuous test** | GENUINELY_FIXED (R3) |
| FIND-2-2 | process | WARNING | A dead parameter (`boundaryTickSeq`) threaded through but never used | GENUINELY_FIXED (R3) |
| FIND-2-4 | domain | NOTE | Cold-boot semantics on a NULL last-charged-date were undefined → ratified by **D76(1)**: fire only the current boundary, no multi-day catch-up | GENUINELY_FIXED (R3) |
| FIND-2-5 | process | NOTE | Event-line wire shape (envelope prefix vs. field) underspecified → ratified by **D76(2)** | GENUINELY_FIXED (R3) |

Note FIND-2-1: a test that *looks* like a negative test but never executes the
call it claims to exercise is worthless — and the reviewer caught it because the
process reviewer's job includes "if you deleted the new code, would this test still
pass?" Wiring the `Replay()` call so the test's stated anti-fake intent became
load-bearing is **helper-wiring, not assertion-weakening** — the implementer is
allowed to strengthen the test's setup, never to soften its assertions.

### Rounds 3–4 — the tail of real-but-smaller findings

Replay-test wiring landed, a dead parameter removed, stale comments corrected.
These are `WARNING`/`NOTE`-class — and under the **zero-tolerance exit, NOTEs
count**. A round does not end because "only NOTEs remain."

### Round 5 / Step 5.5 — THE ESCAPE: phantom-in-production

R1–R4 converged clean on the code the reviewers were reading. Then the **Step 5.5
production wiring audit** — a *fresh* agent that had seen none of the prior rounds
— traced the rate adapter from the production composition root and found:

> the simulated rate-source adapter returned "no rate" for **every production
> lookup**. Rates were populated *only* by a test-only install seam
> (`//go:build integration`). In the shipping binary the feature was a **no-op** —
> while every E2E test was green.

| ID | Reviewer | Severity | Issue (one line) | Status |
|----|----------|----------|------------------|--------|
| FIND-5-1 | wiring audit | BLOCKER | The simulated rate-source adapter's production path returned an empty store; real values existed only behind a test-only seam. Its docstring promised "generated rates." | GENUINELY_FIXED (R6) |

**This is the phantom-in-production class, and it is why this document exists.**
Every unit and E2E test passed. The adapter was instantiated, exported, called.
But the *production* path of its primary method returned an empty store; the only
place real values ever appeared was a `//go:build integration` test seam that
populated it. Ship this and overnight financing silently does nothing, forever —
green in CI, inert in production. Worse, the docstring *claimed* it generated
rates, so the next reader would trust a contract the code did not honor.

The fix (R6): build the **real generator** on the production path — per-instrument
carry-aware annual baselines, ACT/360, the 3× rule pre-multiplied into the stream,
banker's-rounded, unknown instrument → fail-closed error. No float, no wall-clock,
no RNG: a pure (instrument, value-date) function, so it stays deterministic. The
generator then *surfaced a latent boundary-definition bug* (positions opened before
the engine's first boot boundary were being charged) — caught and fixed via the
held-at-instant filters. One real fix exposing the next is the loop doing its job.

### Rounds 6–9 — hardening the now-live path

With real values flowing, new tests became meaningful: a generator-path determinism
test (no install override), the boundary-filter cases, conservation, cross-account
isolation. R8 caught a tenant identifier leaking into a warning log (a security-auditor
finding — sensitive identifier in log output) and missing provenance comments on
shared helper extractions. All `GENUINELY_FIXED`.

### Round 10 — the zero round (it requires MORE justification than findings)

All three reviewers reported **zero findings on the same round**. Under the rules,
that clean pass is the *suspicious* outcome and demands more evidence than a round
full of findings: each reviewer had to justify the clean call per-axis with
specific traces. The **Step 5.5 wiring re-audit** then re-confirmed the generator
was live in the shipping binary — verified mechanically: the symbol-table check
showed the production generator linked and the test-only install seam **absent**
from the production binary. Only then did **Step 6** run the full CI gate green
(including the ~19-minute serial determinism suite proving byte-identical rollover
across boots) and the slice ship.

---

## The escape and the rule it became

The empty-store adapter slipped past the **test writer** (the tests it wrote were
satisfiable by the test-seam-populated store), the **implementer** (who wired a
real-looking adapter with a real-looking docstring), and **all three R1–R4
reviewers** (who read code that compiled, was instantiated, and passed every
test). It was caught only by the **fresh Step 5.5 wiring audit** — the last line of
defense, and the one step the workflow makes mandatory precisely because this
failure mode is the most common one.

Per **Section G — every escape becomes a permanent rule** — this one became
several, folded into the v2 revision so the catch moves from Step 5.5 (last line)
to Round 1 (first reviewer):

- **Forbidden Pattern 12 — phantom-in-production adapter.** A simulated/stub
  adapter intended to ship MUST GENERATE real values on the production path.
  "Simulated ≠ inert." An adapter whose primary method returns empty/zero on the
  non-test path is phantom-in-production, not a stub to fill in later.
- **Forbidden Pattern 13 — docstring–code divergence on a load-bearing path.** A
  comment promising behavior the code does not implement is a *correctness*
  finding, not a style nit — the comment is the contract the next reader trusts.
- **The Step 3.5 phantom-adapter grep** — the cheap mechanical gate now confirms
  the composition root instantiates the concrete adapter unconditionally and its
  primary method returns real values on a non-test path.
- **A Phantom-in-Production verification block in all three reviewer prompts** — so
  R1 reviewers, not just the wiring auditor, hunt the empty-store pattern.

And the *other* S8d lesson — that pinning all six axes up front (D70) prevented the
re-loop churn that open axes caused on earlier slices — was generalized into the
**Step 1 Ambiguity Sweep** and its nine-axis checklist. The process converged
instead of repeating the same class of mistake.

---

## What to take from this

- **Rounds and findings are normal.** S8d took 10 rounds and surfaced findings in
  six of them. Across the project the per-slice round count ranged from a single
  round on the smallest slices to twelve on the hardest; a handful of rounds on a
  non-trivial slice is the median, not an alarm. "3 review rounds with 14 findings"
  is the process *working*.
- **A clean Round 1 on a non-trivial slice is the suspicious outcome.** Reviewers
  are rewarded for finding real issues; finding zero is a *failure state* unless
  proven. If a hard slice comes back clean on R1, distrust it before you celebrate
  it.
- **The zero round requires MORE justification than findings.** Ending the loop is
  the high-stakes moment. Every reviewer's clean call had to be defended per-axis,
  and the wiring audit had to be re-confirmed mechanically, before the slice could
  ship. Stopping short of zero — "mostly clean," "only NOTEs left" — is the failure
  the zero-tolerance exit exists to prevent. **A NOTE is a finding.**
- **The fresh wiring audit earns its mandatory status.** The defining defect of
  this slice — a feature green in every test and inert in production — was invisible
  to everyone who had been reading the code and was caught only by the agent that
  came in fresh and traced from the production entry point. Never skip it because
  "the reviews were clean."
- **Every escape becomes a permanent rule.** The empty-store adapter cost a late
  round once. Because it became Forbidden Patterns 12/13, a mechanical grep, and a
  reviewer-prompt block, the next instance of that class is caught in Round 1 by
  cheaper agents. That is the whole point: the process gets stricter at the exact
  spot it was beaten, and it never loses that ground.
