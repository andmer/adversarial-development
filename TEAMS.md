# Scaling to Teams — Lanes, the Steward, and the Merge Queue

*Companion for running the methodology with more than one human/orchestrator
pair. Read `OPERATOR.md` first — every lane still has an operator.*

The kit as written is **single-writer by design**: one ▶ pointer, one frozen
spec with one linearized decision history, one full-gate-then-commit pipeline
onto trunk. That is not an oversight. The serialization is concentrated in
exactly the places where parallelism is most dangerous — decisions and
integration — and some of it *is the product*: "frozen decisions are never
re-litigated" is cheap precisely because the decision history has one writer.

So the team adaptation is not "remove the serialization." It is **partition
the work so the serial points stay serial and everything else runs
concurrently.** Two facts shape everything below:

1. **Within a slice, wall-clock is agent time, not human time.** Two humans do
   not make one slice faster — the loop is sequential by design (tests →
   implement → review rounds → audit). Team scaling means more **lanes** (more
   slices in flight), never faster slices.
2. **The agents already are the team.** A lane is one human/orchestrator pair
   plus the agent crew. Before adding lanes, ask whether you need them — one
   operator with a converged process covers ground that used to justify
   several people.

| Stays serial (load-bearing) | Runs parallel (partitioned) |
|---|---|
| The SPEC + §8 decision log — one writer (the steward) | Slices in disjoint lanes (bounded-context-partitioned) |
| The merge gate — one full-gate run at a time (the queue) | Each lane's inner loop, end to end |
| The zero-tolerance exit, per slice — unchanged | Each lane's ▶ pointer, finding trackers, checkpoint sign-offs |

---

## The Lane Model

A **lane** is a set of bounded contexts plus the slices that touch only them,
owned by one human/orchestrator pair.

**The lane boundary already exists in the kit.** The architecture rule — *a
context never imports another context's domain or adapters, only its public
app API, lint-enforced* — is precisely the property that makes two slices safe
to run concurrently. The methodology built its own parallelization primitive;
teams are just the first consumer of it.

Rules of the lane:

- **Partition by context, not by file.** A lane owns whole bounded contexts.
  Two lanes editing one context is not two lanes; it is one lane with a race
  condition.
- **Cross-lane consumption is frozen-surface only.** A lane may depend on
  another lane's *shipped, drift-guarded* contract (the context's app API, the
  external API spec) — never on its in-flight work. Needing another lane's
  unfrozen surface means you found a **sequencing dependency**: reorder the
  slices, don't parallelize them. Coordination by conversation ("I'll have it
  ready by the time you need it") is coordination by silence — the exact
  failure class the outer loop exists to kill.
- **The PLAN splits its pointer, not its authority.** One global block at the
  top — current spec version, latest decision, lane roster — then a **per-lane
  ▶ Status & Next block** (Done / exact Next task / loop one-liner), each
  updated in that lane's own commits. Shared sections (the carried-debt
  ledger) live once, globally; because they live in one file, the merge queue
  serializes their numbering for free.

---

## The Spec Steward — Serialize the Law

One role — the **steward** — owns every write to the SPEC: version bumps, Dxx
rows, scope changes. It can rotate; it cannot be bypassed.

- **Lanes raise; the steward ratifies.** A lane's ambiguity sweep batches its
  open axes into one questionnaire at slice start — that is the lane's
  synchronization point with the steward. One round-trip, then the lane runs
  autonomously. This is why the sweep is secretly the most parallel-friendly
  rule in the kit: it front-loads a slice's decision traffic into a single
  burst instead of blocking mid-slice.
- **Conflicting raises are serialized, not merged.** When two lanes raise
  overlapping decisions, the steward resolves them in one order; the
  later lane rebases its design on the newly frozen row **before Step 2**.
  A test suite written against a decision that lost the race is rework you
  chose by skipping this.
- **Do not delegate ratification per-context.** It is the tempting fix for
  steward load, and it fragments the single decision history into per-team
  lore — the pre-§8 world with extra files. Rejected. If the steward is
  saturated, you have too many lanes (see the ceiling below), not too few
  stewards.
- **Operator vs steward.** Each lane's operator does everything `OPERATOR.md`
  says — sweep answers, raw-output spot-checks, checkpoint sign-off — *for
  their lane*. The steward ratifies only spec-level decisions. At small scale
  one person holds both roles; that is exactly the single-operator kit.

---

## The Merge Queue — Serialize the Gate

Trunk remains the integration point; long-lived branches remain banned. What
changes: with multiple writers, "run the full gate, see it green, then
commit" is a race — two lanes can each see green against trees that are both
stale the moment the other merges. The fix is the industry's:

- **A merge queue is the gate runner.** Lane work integrates through a queue
  that runs the end-of-slice full gate **serially, against the rebased
  candidate tree** — never against the tree the slice was developed on. One
  slice in the gate at a time (which also kills the container-starvation
  flake class — the kit already bans concurrent heavy suites).
- **The rebased-gate rule (the moving-baseline guard).** The kit's checks
  assume the frozen baseline does not move during a slice; parallel lanes
  move each other's baseline. A slice reviewed clean against last week's
  trunk is **not** clean against today's. At queue time, after rebase:
  1. Re-run **Step 3.5** in full (including the fabricated-green re-run)
     against the rebased tree.
  2. Re-check **Step 1.5's cross-slice arithmetic**: did any lane that merged
     since this slice's review freeze a formula or threshold this slice's
     test setups now interact with?
  3. If the rebase touched the slice's own hunks, or check 2 fires, run **one
     delta-scoped review round** (changed hunks + one call-hop, per the
     existing fix-round rule) before merging. Reviewers review; the lane's
     author never self-clears the delta. Prior findings do not reopen.
- **Lane branches are slice-lived,** hours to a day — they exist so the queue
  has something to rebase, not as a place where work accumulates unreviewed.
  Incomplete user-facing work stays feature-flagged on trunk, exactly as
  before.

The inner loop itself is untouched. Every slice, in every lane, still runs
the full sequence — blind test writer, non-weakening implementer, parallel
independent reviewers, fresh wiring audit, zero-tolerance exit, NOTEs count.
Nothing in this document touches a slice's internals.

**Cross-lane checkpoints.** Per-lane checkpoint audits continue unchanged. At
multi-lane scale, add a periodic **steward-run cross-lane audit**: fresh
agents hunting specifically for cross-lane drift — the cross-slice-arithmetic
class at lane scope, contract assumptions that diverged, the same invariant
defended differently in two contexts. This is the team-scale version of the
checkpoint that has caught drift per-slice review missed.

---

## The Operator Ceiling — Honest Limits

- **One steward sustains roughly 2–3 lanes.** Decision traffic is bursty and
  front-loaded (the sweep batches it), which is what makes even that
  possible. Past it, lanes queue on ratification — and the wrong fixes are
  delegating the law or rubber-stamping raises. The right fixes are fewer
  lanes, smaller slices, or accepting the throughput.
- **Adding humans without adding lanes adds nothing.** A slice's wall-clock
  is the agents'. If two engineers share a lane, one of them is watching.
- **This scales to a handful of lanes, not an org chart.** For a large
  organization, adopt the *shapes*, not the single PLAN file: the inner loop
  maps naturally onto a per-PR pipeline (it is a hyper-rigorous PR process —
  blind tests, independent review, merge-queue gate), and the outer loop is
  an RFC process — serialized decision-making with parallel execution, which
  is how large orgs already work when they work. What does not survive that
  translation untouched is the single ▶ pointer and the one-file PLAN; what
  must survive it is separation of duties, the linearized decision record,
  and the zero-tolerance exit.

---

## The Meta-Rule Audit of This Document

Every adaptation above, classified the way the kit demands:

- **Merge queue / rebased gate** — changes WHEN and against WHAT TREE checks
  run, and HOW DEEP (one delta-scoped round, by risk). Acceptable.
- **Per-lane ▶ pointers, lane-scoped trackers** — changes HOW MUCH
  bookkeeping surface each lane re-reads. Acceptable.
- **The steward** — changes who *decides*, which was always a human and is
  now a named human. It does not change who *validates* work: no author
  reviews their own slice, no lane self-clears its rebase delta, no
  orchestrator skips independent verification, no NOTE becomes waivable.
  Acceptable.

Anything a team proposes beyond this document gets the same classification —
by effect, not description. Parallelism pressure is efficiency pressure, and
"we're a team, we can't afford the serialization" is "it only saves a round"
wearing a jersey.
