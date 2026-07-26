# Standing Obligations — Closing the Absence Class

**Date:** 2026-07-25
**Status:** Design, approved for planning
**Target:** template v3
**Files affected:** `build-methodology.md`, `adversarial-development-workflow-template.md`,
`OPERATOR.md`, `README.md`, `TEAMS.md`, `CHANGELOG.md`

---

## 1. The defect class

A defect where nothing written is wrong — something unwritten is missing.

The standard verification toolkit is a set of closed loops. Each technique
compares one artifact against another artifact derived from the same
understanding:

| Technique | What it compares | Why an absence escapes |
|-----------|------------------|------------------------|
| Tests | code ↔ tests | both written from the same design doc |
| Coverage | lines ↔ execution | a missing line has no coverage to lack |
| Mutation testing | code ↔ mutated code | you cannot mutate code that isn't there |
| Type systems | declarations ↔ uses | consistency of what's written |
| Wiring audits | entry point ↔ components | traces what exists |
| Code review | diff ↔ reviewer's model | an absence isn't in the diff |

Every one finds contradictions *inside* the system. An absence is not a
contradiction; it is a gap outside the loop entirely.

The formal frame is **safety versus liveness**. Safety is "nothing bad happens"
— violated by a finite trace, so a test can witness it. Liveness is "something
good eventually happens" — testable only if someone names the good thing. Test
suites accrete safety properties naturally, because they are cheap and always
terminate. A mature suite therefore drifts toward being comprehensively
safety-checked and structurally blind to liveness.

The reference escape (`D178`): conservation held, ledger balance held,
determinism held, isolation held — and the resting order silently stopped
firing. `replace` was correct in S4 when nothing fired. The pump was correct in
S6c. Their product belonged to nobody.

### 1.1 Where this kit closes the loop today

The gap is narrower and more precisely located than "all verification is
closed." Three specific sites:

- **Step 1.5 checks 1–4** enumerate "every data flow in the design doc" and
  demand a test per flow. The universe of things to cover is read off the
  design doc, so an obligation from a previously shipped slice is not in the
  enumeration at all. It is not missed — it is out of scope by construction.
- **SPEC §10** holds `AC1…ACn` scoped to *the current thin vertical slice*, and
  traceability runs one direction only (README: "every code change traces to an
  AC"). The arrow never reverses, so a live AC from S4 is never asked about
  during S6c.
- **SPEC §7** (Boundaries — what NEVER happens) and **SPEC §11** (Permanent
  non-goals — true NEVER) are both safety-shaped. There is no section anywhere
  in the SPEC layout for "what must keep happening." The liveness blindness is
  visible in the section list itself.

### 1.2 The mechanism already exists once

TEAMS.md's **frozen-since register** (`FRZ-N`) is this exact machine, restricted
to the frozen-arithmetic class and justified only at team scale: an append-only
table of prior commitments, greppable keys including data-shape triggers, cost
bounded by `merge-base`, `KEYLESS` rows failing closed to judgment.

This design generalizes it from *frozen formulas* to *live obligations* and
brings it to single-operator scale. It is not a new mechanism to invent and
defend; it is an already-argued mechanism widened, and it inherits the
cost-bounding design rather than re-deriving it.

### 1.3 The cost asymmetry, and its escape

Frozen-since bounds cost to interim rows since `merge-base` because its question
is interim-scoped: *"did anything freeze since my baseline?"* The obligation
question is full-history by nature: *"does my new mechanism break anything ever
shipped?"* A naive cross-product is |obligations| × |mechanisms|, growing
forever.

The escape is the kit's standard move, stated in TEAMS.md: **turn the judgment
scan into a grep plus a bounded judgment on the hits.** With greppable keys per
row, the cross-product becomes a mechanical grep over the slice diff, and only
hits get judgment. N×M collapses to a sweep plus a handful of verdicts, and full
history stays affordable.

**The residual risk is sharp and must be named in the shipped docs: liveness
obligations are disproportionately likely to be `KEYLESS`.** "The stop-loss must
still fire" has no symbol to grep — which is precisely why `D178` escaped. In
the frozen-since register `KEYLESS` is a rare edge case; here it is the common
case for the highest-value rows. The `KEYLESS` → forced-judgment rule therefore
carries far more weight in §14 than it does in `FRZ-N`, and the docs must say so
rather than inheriting the framing silently.

---

## 2. Design decisions

| # | Decision | Rejected alternative | Why |
|---|----------|---------------------|-----|
| 1 | Register holds **liveness-shaped rows only** | Promote every live `AC` | ~80% of ACs restate what tests, types, and reviewers already catch. Promoting all of them dilutes the register and scales per-slice cost with the safety surface the loop already handles well. |
| 2 | Rows are **seeded mechanically, curated by the operator** | Blank-page operator authoring | A blank page under-covers everything already shipped. The seed is derivation from inside the loop and buys only the blank page — independence comes from the curation verdicts, not the seed. |
| 3 | Register lives in the **SPEC**, not the PLAN | PLAN, next to the debt ledger and `FRZ-N` | The debt ledger and frozen-since register are derived from work in flight. Obligations are the opposite: non-derivable, externally authored, permanent until retired. Putting them in the law routes mutation through §8. |
| 4 | Sweep runs at **Step 1.5 and Step 5.5** | A dedicated QA seat (8th role) | Matches the phantom-in-production precedent: the kit's answer to a whole escape class was a forbidden pattern + a Step 3.5 grep + reviewer-prompt blocks — instrument existing steps, multiply catch points, hold the roster. |
| 5 | Sweep runs at **both**, not Step 5.5 alone | Step 5.5 only | A `BROKEN` verdict after zero-tolerance exit is the most expensive place to discover it — full re-implement plus re-spawn all reviewers. The kit deliberately shifted phantom catches earlier for this reason. |
| 6 | Cut as **v3** | Dated sub-entry under v2 | TEAMS.md and Step 0.5 landed as v2 sub-entries because neither changed the inner loop. This changes Step 1.5, Step 5.5, and Step 6. |

### 2.1 Why not a dedicated QA seat

The original framing was a dedicated QA seat on every slice. Two objections
carried:

**An agent seat partly recreates the loop it exists to break.** An agent
authoring the obligation record derives it from the design doc and the code —
the exact operation the pattern says is impossible. The non-derivable input has
one source in this kit: the operator. `OPERATOR.md` already formalizes operator
engagement as numbered moments; obligation curation is a sixth moment. An agent
or the orchestrator does only the mechanical part — the sweep and the totality
check.

**Step 5.5 is the seat, pointed in only one direction.** The wiring audit asks a
fresh, uncontaminated agent: *"is the new code reachable from production?"* The
exact mirror is *"is the old obligation still reachable from production?"* Same
freshness requirement, same call-chain tracing method, one more section in an
existing prompt. `D178` is a wiring audit run backwards.

This also drops the cost from "a seat per slice, forever" to "a grep per slice
plus occasional operator writing," which materially changes the Section F
economics.

---

## 3. The artifact — SPEC §14

A new SPEC section, numbered **§14** (the documented layout runs 1–13; §14 is
free). The section-set is explicitly described in `build-methodology.md` as "an
example — adapt the section set to your system," so the number is soft and the
name is load-bearing.

**Its conceptual pair is §7 and §11.** Both are safety sections — what never
happens. §14 is their missing twin — what must keep happening. Stating that
adjacency in the shipped doc is most of the teaching: a reader who sees two
NEVER sections and no ALWAYS section understands the gap without further
argument.

### 3.1 Row format

```
## 14. Standing obligations (liveness register)

| ID   | Owed to the user (what must KEEP happening)  | Since   | Keys / triggers                    | Proof              |
|------|----------------------------------------------|---------|------------------------------------|--------------------|
| OB-3 | A resting stop-loss fires when price crosses | S4, D61 | KEYLESS — any change to order      | TestStopFiresOn    |
|      | its trigger                                  |         | lifecycle, replace, or match loop  | Cross              |
| OB-7 | Every fill emits a ledger entry              | S2      | ledger., emitFill, OnMatch         | TestFillLedgers    |
```

Column contracts:

- **ID** — `OB-n`, stable and immutable, cited by handle in commits, findings,
  audits, and decisions. Never referred to by prose phrase.
- **Owed to the user** — phrased as a liveness claim: something that must keep
  happening, in the user's terms. Not an implementation statement.
- **Since** — the slice and (where applicable) the decision that created the
  obligation.
- **Keys / triggers** — the load-bearing column, same contract as `FRZ-N`: the
  symbols, config fields, event types, and table columns that would appear in
  code interacting with the obligation, **plus data-shape triggers** ("any
  change to the order lifecycle"), because a change can break an obligation
  without ever naming it. A row with no adequate greppable key is marked
  `KEYLESS`.
- **Proof** — the name of the check that currently witnesses the obligation.
  **It must enter through a declared production entry point (PROJECT_CONTEXT
  "Production Entry Points") and assert only on user-observable output.** A
  test that constructs the scenario mid-system does not qualify, however
  thorough it is. See §3.3.

### 3.2 Governance

- **Mutation is a versioned SPEC revision plus a `Dxx` row.** Retiring or
  rewording an obligation requires a dated decision carrying rationale and
  rejected alternatives, exactly as any other §8 change.
- **`OB-n` joins the shared decomposition vocabulary** in README, alongside
  `Mx` / `Fx` / `Sx` / `Dxx` / `ACn` / `Rx` / `FIND-<round>-<n>` / `DEBT-N`.
- **At team scale the steward owns §14**, as with the rest of the law. No new
  serialization mechanism is needed; TEAMS.md's spec-steward role already
  serializes SPEC writes.

### 3.3 The Proof contract — execution through the front door

Every other check in this kit is **analysis of artifacts**: tests analyze
assertions, coverage analyzes execution traces, reviewers analyze diffs, and
the Step 5.5 wiring audit statically traces call chains. A human QA is the only
actor who does something categorically different — **executes the system
through its front door and observes the output.** That difference is the whole
reason a human QA catches this class in one session.

`PROJECT_CONTEXT` already declares the anchor and uses it in only one
direction: *"Production Entry Points — the composition root the wiring auditor
**traces** from."* Nothing in the kit **runs** from it.

This is why the reference escape survived. The S4 stop-loss test entered the
system mid-path — where it had always entered — and stayed green while the S6c
pump rerouted production behind it. A check that enters where a *user* enters
cannot be bypassed that way, not because it asserts better but because it does
not get to choose its own entry point.

**The Proof contract, therefore:**

1. The check invokes a declared production entry point. It does not construct
   the scenario at an interior seam.
2. It asserts only on output a user could observe.
3. It runs against the production composition root, with no test-only build
   tags or seams (the same standard the Step 3.5 phantom-adapter grep already
   enforces).

A row whose only available proof fails this contract is marked
`PROOF-INTERIOR` and is **treated as an unproven obligation** — it still gets a
Step 5.5 verdict, and closing the gap is tracked as `DEBT-N`. This is
deliberately visible rather than silently tolerated: an interior proof is the
exact shape of the artifact that let the reference escape run for two months.

This is §6.1 (method diversity) instantiated as the register's core mechanic.
Static tracing and dynamic execution fail differently; §14's value comes from
requiring the one method the kit otherwise lacks.

### 3.4 Seeding

Mechanical seed, run once at adoption and again at each checkpoint audit:

1. Grep the shipped test suite for liveness-shaped assertions — tests whose
   names or assertions take the form "X fires / emits / is produced / is
   scheduled / is delivered."
2. Propose one candidate row per distinct claim, with keys drafted from the
   symbols the test touches.
3. The operator issues **ADD** or **REJECT** per candidate row.

**The docs must state plainly that the seed is derivation from inside the
loop.** It does not confer independence. It kills the blank page for
already-shipped work and nothing more. Independence comes from the operator's
ADD/REJECT verdicts and from the rows they add that no test suggested.

---

## 4. The gates

### 4.1 Step 1.5, new check 7 — Standing-obligation sweep

Extends the existing list (1–4 coverage, 5–6 conformance) with a third
category. Check 7 belongs to the same family as check 6, which already reaches
backward into already-shipped frozen formulas.

For every §14 row:

1. Grep the slice's design doc and its Step-2 tests for the row's keys.
2. `KEYLESS` rows skip the grep and take a forced judgment call against the
   slice's mechanisms — fail closed: if it is unclear whether the slice
   interacts with the obligation, treat it as a hit.
3. Every hit must have a test covering the obligation **before Step 3**. If it
   does not, re-spawn Step 2 with the obligation text and the reason it is
   threatened.

Record the sweep result — every row, hit or no-hit — so Step 6 can verify
totality.

This connects cleanly to the Ambiguity Sweep's **axis 5 (Cadence / when-fires)**,
which already asks the liveness question for *this* slice. Check 7 is axis 5
pointed at every prior slice.

### 4.2 Step 5.5, new prompt section — Standing-Obligation Re-Verification

The Step 5.5 wiring-auditor prompt gains a section. Same fresh agent, same
call-chain method, opposite direction.

**Step 5.5 derives its own scope. It does not inherit Step 1.5's hit list.**
The auditor re-verifies:

- (a) every row Step 1.5 flagged as a hit, **plus**
- (b) every `KEYLESS` row, unconditionally, **plus**
- (c) every row whose keys hit against **the actual diff** (Step 1.5 grepped the
  design doc and the Step-2 tests — intent; Step 5.5 greps what was really
  built).

The reason is the method-diversity rule in §6.1, applied to this design. If
Step 5.5's scope were set by Step 1.5's verdicts, the two gates would share a
single point of failure: an obligation whose keys were written too narrowly is
missed at 1.5 and then never examined at 5.5. Two gates, one blind spot. The
`KEYLESS` clause matters most — since `KEYLESS` is the common case for
high-value liveness rows (§1.3), clause (b) alone recovers the highest-risk
rows regardless of how well anyone drafted the keys.

For each row in scope: trace it from a production entry point to an observable
output **in the post-change tree**. Verdict per row:

- `STILL_FIRES` — the trace is complete in the post-change tree and the auditor
  can name the call chain.
- `BROKEN` — the trace is severed; name the severing change.
- `UNPROVEN` — the auditor could not establish the trace either way.

`BROKEN` and `UNPROVEN` route back to the implementer exactly as
`PHANTOM_WIRED` does, and Steps 4 / 4.5 / 4.6 / 5.5 re-run until clean.
`UNPROVEN` exists specifically so an auditor cannot discharge a row by failing
to find evidence — absence of a trace is not evidence of a trace.

The auditor remains strictly read-only, per the Section C READ-ONLY MANDATE. It
reports; the orchestrator owns any mutation-proof.

### 4.3 Step 6, totality gate

Three mechanical assertions before the final commit:

1. Every §14 row was swept at Step 1.5 — the sweep record is complete, no row
   unexamined.
2. Every row in the Step-5.5 scope (§4.2 clauses a, b, and c) carries a verdict.
3. Every §14 row's **Proof** column names a test that exists and runs.

**Stated limit, shipped in the doc:** the gate proves the register was covered
and that each entry names a real test. It cannot prove the test proves the
thing. That judgment is the operator's, at Moment 6, against the test's
anti-fake property. Without this stated explicitly, the totality gate decays
into exactly the unfalsifiable prose it replaced.

**The concrete failure this admits.** Against the reference escape, assertion 3
would have passed. The S4 stop-loss test still existed, still ran, and still
passed after S6c — it constructed the trigger scenario directly, so it kept
proving the component fired while no longer representing the production path
the pump had rerouted. A green totality gate is compatible with a broken
obligation. What catches that case is Step 5.5, and specifically the fact that
Step 5.5 does not run the test: it traces the call chain in the post-change
tree, and a test that bypasses the pump cannot satisfy that trace. The catch
depends on the two checks using **different methods** — assertion-execution
versus call-chain-tracing — which is §6.1 stated as a design property of this
mechanism rather than as advice.

---

## 4.4 Checkpoint audit — full re-verification (the latency bound)

Checks 4.1–4.3 are **threat-scoped**: they examine the obligations *this slice
appears to threaten*. That leaves an unbounded failure mode. If a slice breaks
an obligation and the sweep does not flag it — narrow keys, a `KEYLESS`
judgment that guessed wrong — no later slice re-examines that row either,
because no later slice threatens it. The obligation stays broken forever and is
found by accident. That is precisely the reference escape's two-month
survival, and threat-scoping alone does not fix it.

A human QA does not ask "what did you change?" before a regression pass. They
re-run everything.

**At each slice-complete checkpoint audit, execute every §14 Proof.** All rows,
independent of what the slice touched. This is cheap — Proofs are executable by
the §3.3 contract, so this is a suite run, not a judgment sweep — and it is the
mechanism that converts unbounded detection latency into a bound:

| Configuration | Worst-case survival of a broken obligation |
|---------------|--------------------------------------------|
| No register (today) | Unbounded — found by accident |
| Threat-scoped sweep only (§4.1–4.3) | Unbounded, if the sweep misses once |
| **\+ full re-verification at each slice-complete** | **One slice** |

The checkpoint audit is the right home and needs no new machinery: it already
fires at each slice-complete, already spawns fresh auditors that did not build
the work, already requires human sign-off, and has already caught cross-slice
drift that per-slice review missed. A `PROOF-INTERIOR` row cannot be discharged
here — it has no executable front-door proof, so it takes a judgment verdict
and stays visible as `DEBT-N` until a real proof exists.

## 5. OPERATOR.md — Moment 6

A sixth engagement moment, at slice sign-off:

- Rule ADD/REJECT on each seeded candidate row.
- Add the obligations no test suggested.
- Spot-check one `STILL_FIRES` verdict against the named test's anti-fake
  property — does the test actually witness the obligation, or does it pass for
  an unrelated reason?

Paired entry in "The Operator's Own Anti-Patterns":

> **Confirming every seeded row and adding none.** A register that only ever
> agrees with the test suite has quietly become a view of the test suite —
> which puts it back inside the loop it exists to break. The rows that matter
> most are the ones no test suggested.

---

## 6. The three companion rules

### 6.1 B — Method diversity in layered verification

**Home:** Section D (the roster table's explanatory paragraph) and Section E
(anti-pattern list).

Section D currently claims the reviewers "check different axes." Three seats
with three prompts does not guarantee three *methods*. The evidence: one
finding was withheld from three reviewers to test whether their check was
sound. Three agents independently confirmed a claim that was false, and a
purpose-built AST tool missed it a fourth time. All four classified SQL by
looking for a statement keyword; the site assembled a keyword-free fragment.
Four independent agents, one shared method. The evidence that moved the
question came from a different method entirely — a reviewer who found `pred +=`
by looking at accumulation patterns rather than keywords.

Two rules:

- **Agreement among same-method checks is not evidence.** Four keyword-based
  surveys are one survey run four times; the agreement reads as confidence
  while contributing zero information.
- **When folding an escape into a new check under Section G, name the method
  the check uses and confirm it differs from the methods already deployed
  against that class.** Vary the mechanism, not just the reviewer.

### 6.2 C — Derivation over assertion

**Home:** canonical statement in `build-methodology.md` Part 3, next to the
convergence material; pointer from Section G.

The principle the kit already applies repeatedly and has never named:

> **Convert assertions into derivations, or into checks.** Don't state the
> count — derive it from the files. Don't assert "no bypasses exist" — count
> them and ratchet the count downward. Don't store the flag — derive it from
> the state and delete the column, so consumers cannot read it. A convention is
> a claim about future behaviour that nothing falsifies until it is violated; a
> structure makes the violation either unrepresentable or loud.

Observed instances, all the same shape — a claim that cannot be
wrong-detected:

- a stored copy of a value derivable from another field, drifted twice, both
  times a shipped risk hole;
- one fact in four representations, drifted, hiding four open bugs under a
  "Retired" heading;
- a convention ("no production code assembles SQL by string concatenation")
  with four independent checks, all wrong;
- a README census, stale four times, each fix buying one round;
- a comment claiming path containment the code did not implement.

Naming the generative rule turns the kit's list of clever instances (the
phantom grep, greppable forbidden patterns, `FRZ-N`, §14 itself) into something
a reader can apply to a case the kit did not anticipate.

**§14 is this principle applied to the one thing that genuinely cannot be
derived.** You cannot compute "what the user is owed" from the code — it is
real external information. Where derivation is impossible, the closest
structural substitute is: have an independent party state it, commit it, and
gate on its totality.

### 6.3 D — Vocabulary redefinition must enumerate citing artifacts

**Home:** `build-methodology.md` §8 "Rules that bite."

> **A decision that redefines an existing term must enumerate the artifacts
> citing it.** The enumeration is part of the row. A `Dxx` that redefines a
> word silently inverts the meaning of every artifact that cited the old
> definition — and the inversion is invisible, because nothing in either
> artifact changed.

Observed instance: a `Dxx` redefined the word "terminal"; a catalog entry
citing the old meaning silently inverted, in the dangerous direction.

This is the direct protection against §14 catalog rot, and it generalizes
beyond §14 to every artifact in the kit that cites spec vocabulary.

---

## 7. Honest limits (shipped in the docs, not just here)

- **The delta depends on the operator's imagination; only the sweep is
  mechanically complete.** A brand-new action nobody thinks to catalog stays
  uncatalogued.
- **The seed is inside the loop.** Test-seeded rows are derived from the same
  understanding as the code. The seed solves the blank page, not independence.
- **`KEYLESS` is the common case here, not the edge case.** Liveness
  obligations frequently have no greppable surface — which is exactly why they
  escape. The fail-closed judgment path carries more weight in §14 than in
  `FRZ-N`.
- **The catalog can rot.** §6.3 is the mitigation, not a guarantee.
- **The totality gate proves coverage, not correctness.** Every row swept,
  every hit verdicted, every Proof naming a real test. It cannot prove the test
  proves the thing — and against the reference escape it would have gone green
  (§4.3).
- **Key quality is written before the interaction exists, and hindsight
  flatters it.** `OB-3`'s keys in §3.1 are drafted as a data-shape trigger wide
  enough to catch the S6c pump — drafted, that is, by someone who already knew
  the answer. An operator writing that row during S4 has no reason to reach
  that wide. This is why §4.2 clause (b) re-verifies every `KEYLESS` row
  unconditionally rather than trusting the keys, and why key quality is
  explicitly a convergence target: when an interaction slips through because
  the keys missed it, that is an escape — widen the row and fold the lesson in,
  exactly as `FRZ-N` does.
- **This design does not catch the escape that produced it.** It catches its
  class. `D178` is what *creates* `OB-3`; nothing here manufactures an
  obligation nobody has written down. That is the same relationship
  phantom-in-production has to Forbidden Pattern 12, and it is the convergence
  contract working as specified — but it must not be described as "this would
  have caught `D178`," which is false.
- **It costs a grep per slice plus operator writing time** — plus a real
  one-time cost that the original framing hid: **building front-door proofs for
  the already-shipped obligations.** Most existing suites enter mid-system, so
  most seeded rows will start `PROOF-INTERIOR`. That backlog is the honest
  price of admission, and it is paid once per obligation, not per slice.
- **`PROOF-INTERIOR` makes the gap visible, not closed.** A row with no
  front-door proof is tracked and judged, never silently accepted — but a
  register full of `PROOF-INTERIOR` rows is a register that has named the
  problem and not yet solved it. The count of such rows is the correct health
  signal, and it should ratchet downward (§6.2, derivation over assertion).

The pattern is worth it exactly when the cost of a silent absence exceeds that
overhead — which, for a stop-loss that does not fire, it plainly does. This is
the same trade Section F already frames for the loop as a whole.

---

## 8. Meta-rule audit of this design

Per Section G, every addition is classified by effect:

- Adds a check; removes none. Changes **WHEN** checks run (a new sweep at 1.5,
  a new section at 5.5) and **HOW MUCH** surface is examined (backward into
  shipped obligations).
- Does **not** change **WHO validates** — no author validates their own work;
  the Step 5.5 auditor remains fresh and read-only; the operator authors the
  register and does not implement.
- Does **not** change **WHETHER the orchestrator independently verifies** — the
  Step 6 totality gate is orchestrator-run against the sweep record and the
  test suite, not against a subagent's summary.
- Does **not** change **WHETHER NOTEs can be waived** — `BROKEN` and `UNPROVEN`
  route back exactly as `PHANTOM_WIRED` does. No new waivable severity is
  introduced.

Classification: **addition, not erosion.** Accepted.

---

## 9. Change inventory

| File | Change |
|------|--------|
| `build-methodology.md` | §14 added to the SPEC layout with its §7/§11 pairing; §8 "rules that bite" gains the vocabulary-redefinition rule; Part 3 gains the derivation-over-assertion principle; **Checkpoint Audits gains full §14 re-verification at each slice-complete (§4.4)** |
| `adversarial-development-workflow-template.md` | Step 1.5 check 7; Step 5.5 prompt section; Step 6 totality gate; **PROJECT_CONTEXT "Production Entry Points" reworded — it is what checks are *run from*, not only *traced from* (§3.3)**; Section D method-diversity paragraph; Section E anti-pattern entries; Section G pointer to the derivation principle and the method-naming rule |
| `OPERATOR.md` | Moment 6; the paired operator anti-pattern |
| `README.md` | `OB-n` in the decomposition vocabulary; one line on the absence class in the two-loops framing |
| `TEAMS.md` | One paragraph: §14 is steward-owned law; its relationship to `FRZ-N` |
| `CHANGELOG.md` | v3 entry, with `D178` as the driving escape |
