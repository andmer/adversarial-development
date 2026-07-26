# The Operator's Manual

*For the human running the process — not the agent. The other two documents are addressed to the LLM; this one is addressed to you.*

The agents do the work. You are the constitution's enforcement mechanism. Every anti-erosion defense in `build-methodology.md` and `adversarial-development-workflow-template.md` — the zero-tolerance exit, the read-only reviewer mandate, the fabricated-green guard, the meta-rule — ultimately backstops to one actor: **the operator notices.** You are not a spectator (the loop runs without you narrating it) and not a micromanager (you do not write code or tests, and you do not re-review what the reviewers reviewed). You are four things: the **ratifier** of decisions, the **spot-checker** of raw output, the **person who refuses** the shortcut, and the **author of what users are owed** — the one input this process cannot derive from itself. When the agent is at its most persuasive, you are the only thing standing between "it only saves a round" and a defect that ships. Read this before you run your first slice.

---

## The Six Moments You Must Engage

The process is mostly autonomous. It touches you at exactly six points. Everywhere else, stay out of the way. At these six, engage fully — a rubber-stamp here is the failure mode the whole apparatus exists to prevent.

Moment 6 is different in kind from the other five. In those, you are checking the agents' work. In Moment 6 you are supplying information the agents **cannot obtain** — what your users are owed. Every other input to this process is derived from the process; that one is not, and no amount of agent rigor substitutes for it.

### 1. The Ambiguity Sweep (Step 1 / pre-slice questionnaire)

Before any test is written, the agent walks the 9-axis checklist and raises every open load-bearing axis at once. **Answer decisively.** Each answer pins an axis so the implementer never has to guess.

- **A good answer pins the axis.** "Balance is signed from the trader's perspective — debits negative, credits positive." "Cold-boot with no prior row seeds zero, never errors." "The adapter generates the value on the production path; it is not test-populated." One sentence, unambiguous, no residue.
- **A bad answer is "whatever you think is best."** That phrase silently converts a STOP into a guess. The agent stopped *because the axis is undefined*; handing it back unresolved tells it to invent the requirement — the exact failure (decisions made in silence) the outer loop exists to contain. Never do this on a load-bearing axis.
- **Deferring is legitimate — but only explicitly.** You may decline to pin an axis the slice doesn't critically need. When you do, it becomes a **DEBT-N** item in the carried-debt ledger: your deferral, recorded, with a stable handle. A deferral by silence is not a deferral; it is a guess you didn't notice you authorized.

Every axis you pin here is one fewer implement → review → fix round later. This is the cheapest leverage you have all slice.

### 2. §8 Decision Ratification

When the agent hits an undefined load-bearing behavior, it stops and raises a decision. Your job is to respond with **enough rationale that the Dxx row can record why-not the alternatives.** "Use X" is not ratification — it produces a row that is not an ADR. "Use X because Y; we reject Z because it forecloses the latency target we committed in §1" is ratification — it produces a row future-you cannot re-litigate.

Insist the agent present the decision in **plain language**, not codenames and IDs. The trail (Dxx, version bump) goes in the written row; the question you answer should be in plain words. If you can't follow the question, you can't ratify it — you can only rubber-stamp it.

**A decision you ratify in 10 seconds without reading is a silent decision with extra steps.** The §8 log's entire value is that nothing load-bearing got decided without a record *and a human who understood it.* A row you didn't read is a silence the process now believes was a decision.

### 3. PROJECT_CONTEXT / Roster Confirmation (Bootstrap)

At Step 0, the agent proposes a filled-in PROJECT_CONTEXT block and a specialist roster. Most of it is mechanical and inferred correctly: tech stack, test commands, entry points. **Rubber-stamp the mechanical fields. Read the nuanced ones.**

The low-confidence guesses hide in exactly four places: **forbidden patterns, the concurrency model, the time/ID policy, and the test-discipline invariants** — plus the **specialist roster**. These are where the agent is inferring judgment it doesn't have, and a wrong value here propagates into every subagent spawn for the life of the project (every reviewer receives this block verbatim). The agent flags its low-confidence fields; read those even if it doesn't. For the roster: confirm, trim, or defer each proposed specialist — the no-empty-agent rule means an agent with no project-specific judgment shouldn't exist, so an over-eager roster is a real cost, not a free one.

### 4. Checkpoint Sign-Offs

At Foundation-complete, the first risk-gated/critical-path slice, and each slice-complete, the agent runs an independent audit and asks for your sign-off. **This is a gate, not a ceremony.** It has caught cross-slice arithmetic drift that per-slice review missed. A sign-off you give without looking is the ceremony it was designed not to be.

Before you sign, look at exactly three things:

- **The finding-tracker summary** — rounds, total findings, final status of each. A slice that converged in one clean round on a non-trivial change deserves a second look, not a faster signature.
- **The carried-debt ledger delta** — what new DEBT-N items appeared, and whether each is genuinely cosmetic or forward-bound. At a checkpoint the default is **fix-now over defer**; a debt item carried by silence rather than explicit inconsequentiality judgment is a deferral you didn't make.
- **One raw reviewer output** — not the orchestrator's summary of it. Pick one and read it (see Moment 5).
- **The full standing-obligation re-verification result.** At each slice-complete checkpoint, *every* §14 proof is executed — not just the ones this slice looked like it threatened. This is the regression pass, and it is what bounds how long a silently-broken obligation can survive to one slice. A checkpoint that re-verified only the threatened rows has left the bound unbounded.

### 5. Spot-Checking Raw Reviewer Output

The documents name **orchestrator cheating** — summarized output that hides problems, a skipped reviewer, a weakened prompt — as an anti-pattern whose only defense is "the user spot-checks raw reviewer output." **That defense is you. It is your standing job, not an occasional courtesy.**

The orchestrator is the weakest link by its own admission; its completion bias will tempt it to summarize a reviewer's three findings as "minor issues, addressed" and move on. The only thing that catches a softened summary is reading the reviewer's actual words. Frequency guidance:

- **Every checkpoint** — read at least one raw reviewer output in full.
- **Any round that smells too clean** — a first-round zero-findings pass on a non-trivial change is the single highest-value thing to read. Zero findings requires *more* justification than findings do, per the prompts; verify the justification is real and not "looks good to me" in formal clothing.
- **Randomly otherwise** — unpredictability is the point. If the orchestrator can't predict when you'll read raw output, it can't safely cut a corner anywhere.

You are looking for: findings that exist in the raw output but not the summary; a reviewer that was spawned but returned nothing of substance; a "GENUINELY_FIXED" the reviewer didn't actually justify; a NOTE quietly dropped instead of fixed.

### 6. Curating the Standing Obligations (SPEC §14, at slice sign-off)

Everything else in this process compares two artifacts derived from the same understanding — tests against code, coverage against lines, reviews against diffs. All of it finds *contradictions inside* the system. None of it can find an **absence**: a behavior that silently stopped happening, where nothing written is wrong and something unwritten is missing. There is no failing test, no red build, no diff to review. Every gate reports green.

The only defense is a record of what users are owed, authored by someone who is not deriving it from the code. **That is you.** An agent writing this register would compute it from the design doc, which puts it back inside the loop it exists to break.

At each slice sign-off, three things:

- **Rule ADD or REJECT on each proposed row.** The agent seeds candidates by grepping the shipped suite for liveness-shaped assertions ("X fires", "Y is emitted"). Those are proposals, not entries.
- **Add what no test suggested.** Ask: *what does a user now depend on that they didn't before?* This is the part no mechanism can do for you, and it is the part that matters most.
- **Spot-check one `STILL_FIRES` verdict** against the named proof's anti-fake property. Does the check actually witness the obligation, or does it pass for an unrelated reason?

**The anti-pattern to watch in yourself: confirming every seeded row and adding none.** A register that only ever agrees with the test suite has quietly become a *view* of the test suite — which is back inside the loop. If a slice ships and you added zero rows, that is a claim that the slice created no new user-visible commitment. Sometimes true. Rarely.

**On `PROOF-INTERIOR` rows.** A proof must enter through a declared production entry point and assert on user-observable output. Most existing tests enter mid-system, so when you first adopt this, most rows will be flagged `PROOF-INTERIOR` — an obligation you have *named* but cannot yet *prove*. That backlog is real work and it is tracked as `DEBT-N`. **Do not let a `PROOF-INTERIOR` row read as covered.** Its count is the single best health number this process produces: how much of what you owe your users is currently provable only from inside the system. Watch it go down.

---

## The Refusals You Will Need to Make

Erosion is always sold to *you*, by a very persuasive agent, as efficiency. The pitch is never "let me skip a safeguard" — it is always "this saves a round." Your classification tool is the **meta-rule**: an optimization is acceptable if it only changes **WHEN** a check runs, **HOW MUCH** surface is re-read, **WHAT MODEL** runs a mechanical step, or **HOW DEEP** verification goes. It is erosion — and you refuse it — if it changes **WHO** validates, **WHETHER** you independently verify, or **WHETHER** a finding can be waived.

**Classify by effect, not by how the agent describes it.** The agent's framing is the thing you are trained to see past. Here are the pitches you will actually hear:

| The pitch | What it actually changes | Your response |
|---|---|---|
| "The reviewers were clean last round — skip the wiring audit this time." | WHETHER independent verification runs. The wiring audit is the last line against phantom-wiring; "clean reviewers" is exactly when phantom wiring survives. | **No.** The wiring audit is mandatory every slice. Clean reviewers are not a substitute for it. |
| "This is hygiene-only, I'll use the lighter gate." (and you haven't seen the diff) | HOW DEEP — *legitimately*, but only if the hygiene-only claim is true. You haven't verified it. | **Show me the diff first.** Hygiene-only is your call to confirm, not the agent's to assert. When in doubt, it is not hygiene-only. |
| "Only NOTEs remain — shall I proceed?" | WHETHER a finding can be waived. A NOTE is a finding. | **No.** Zero-tolerance is absolute. "Only NOTEs left" is "not done." Fix them. |
| "I combined steps 2 and 3 for efficiency since the change is small." | WHO validates. One agent authoring tests and code destroys separation of duties — the author now validates its own work. | **No.** Re-run them separately. The test writer never sees the implementation; that is the whole guarantee. |
| "Same model wrote and reviewed — it's faster." | WHO validates. | **No.** An author never reviews its own work, regardless of speed. |
| "I'll summarize the reviewer output to save you reading." | WHETHER you independently verify. | **No.** Pass raw output through. I spot-check it; that requires it to exist. |
| "This slice doesn't touch any of the §14 obligations — skipping the sweep." | WHETHER independent verification runs. The sweep is what determines that; the agent has substituted its own conclusion for the check. | **No.** The sweep is how you *find out* whether the slice touches them. `KEYLESS` rows fail closed for exactly this reason. |
| "The totality gate is green, so the obligations hold." | Nothing — but it is a false claim. The gate proves each Proof *exists and runs*, not that it proves the thing. | **That is not what green means there.** A proof entering mid-system stays green while production reroutes around it. Show me the Step 5.5 traces. |
| "I added a fifth check for that escape class." (and it works like the four that missed it) | Nothing — it is the *appearance* of convergence. | **What method does it use?** If it fails the same way as the checks that already missed it, it is cost without coverage. |

The first three columns are the discipline. The fourth column — your refusal — is the only one the process can't generate for itself. "It only saves a round" is how erosion is always sold. When you hear it, that is your cue to look harder, not to relax.

---

## Week-One Expectations (Calibration)

The first slices feel slow. That is the process working, not failing. Calibrate now so you don't mistake friction for malfunction and start eroding the very safeguards that are earning their keep.

- **Multi-round slices are normal.** 2–3 review rounds with a dozen findings on a non-trivial slice is the system catching what would otherwise have shipped. A slice that converges instantly on a real change is more suspicious than one that takes three rounds. Do not read round count as a problem to optimize away.
- **Reviewer false positives happen — that is what REBUTTED is for.** A reviewer occasionally flags a non-issue. The remedy is the evidence requirement (file:line + traced call chain) and the implementer's evidence-based REBUTTED case that a reviewer re-evaluates — *not* lowering the severity bar to reduce noise. A reviewer that finds nothing is in a failure state by design; do not "calibrate" it toward silence.
- **Environmental flakes are real — and never suppressed.** Container-heavy suites produce host-load timeouts that are genuinely environmental, not logic defects. The discipline is **isolated re-run root-causing**: run the one test alone, serially — a real defect reproduces in isolation, a parallel-starvation flake passes. Never `skip`, never suppress, never hand-wave. Zero flaky tests is the standard.
- **The slowness is front-loaded on purpose.** The convergence mechanism — every escape becomes a new permanent rule — is what makes later slices faster *without removing any guarantee.* You are paying down a defect class permanently, once. Early friction is the price of never debugging that class again. If you erode the safeguards to go faster now, you forfeit the speedup that was coming for free.

---

## What You Must Never Do (The Operator's Own Anti-Patterns)

The agent has its failure modes; the workflow documents catalog them. These are *yours*. The process cannot defend against the operator — you are outside it.

- **Never waive a NOTE to ship.** A NOTE is a finding. The moment you waive one "just this once," the zero-tolerance exit is no longer absolute, and the agent has learned the bar moves under pressure. It does not move.
- **Never answer an ambiguity sweep with "you decide" on a load-bearing axis.** That is you, the human, manufacturing a silent decision — the exact thing the §8 discipline exists to prevent, committed by the one actor it can't catch.
- **Never edit the spec directly.** Every spec change is a versioned revision + a new dated Dxx row with rationale and rejected alternatives. A silent edit to a frozen contract is the worst possible example to set, because the whole org-chart of agents trusts that the SPEC is law and that law changes only on the record.
- **Never accept a green you didn't see.** When you commit — or authorize a commit — the full end-of-slice gate ran, and *you saw it finish green*, or it didn't run. "The agent said it's green" is a fabricated-green you imported by trust. Re-run it, or watch it run. Never commit in parallel with the gate.
- **Never sign off a slice having added zero standing obligations without asking why.** The register is the one input the agents cannot produce. If you only ever confirm what the seed proposed, you have delegated "what do users depend on" to a grep over the test suite — and the whole point of §14 is that the answer is not in there. Adding nothing is sometimes correct. Adding nothing *by default* is how the register rots into decoration.
- **Never let the agent re-litigate a frozen decision.** When a request smells like a decision already in §8, the answer is to **cite the Dxx row**, not re-argue it. The rejected alternative is usually already recorded with the reason it was rejected. Re-opening a settled decision because the agent surfaced it again is how a frozen contract thaws.

---

## What the Discipline Buys

The first slices are slow and the refusals are tiring. Here is what you are buying, and what it feels like once the process has converged:

- **A cold-start resume in one command.** A fresh session — yours or a new agent's — runs `/next`, reads the ▶ Status & Next block, skims §8 for the *why*, confirms trunk-green, and is working on the exact next task. No re-derivation, no "where were we," no archaeology. The repo is the source of truth; conversation memory is disposable.
- **Decisions that never get re-argued.** Every load-bearing choice carries its rationale and its rejected alternatives, frozen by version. You answer each question once. Six months later, nobody re-opens it — they cite it.
- **A defect class that never recurs twice.** Every bug that escapes all the agents becomes a new permanent rule in the agent that should have caught it. The process converges. The thing that bit you last month is now a greppable forbidden pattern that a reviewer flags automatically, forever.
- **A bound on how long a broken promise can survive.** Without a standing-obligations register, a shipped behavior that silently stops working is found by accident — which can mean months. With §14 swept every slice and every proof re-executed at each slice-complete checkpoint, the worst case is **one slice**. That is the difference between "we hope someone notices" and a number you can state.

That is the trade: inconvenient discipline now, in exchange for a system that gets *faster and safer at the same time* as it grows — because the guarantees never come out, only the friction does. The process is the product. Your job is to refuse every offer to trade the first for the second.
