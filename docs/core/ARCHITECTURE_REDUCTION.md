# Architecture Reduction — what Escapement Core must own

```text
STATUS      review artifact, no implementation
INPUT       landscape review, 2025 - Aug 2026
BASELINE    lambda-RLM / RAH + LangGraph + Parallax + Microsoft AGT
            + Shepherd + Self-Harness / HarnessBank + MCP
QUESTION    given all of that exists, what code and semantics
            does Core still need to own?
ANSWER      three subsystems, one of them mostly already built
```

This document performs the reduction requested before implementation
begins. It does three things: verifies the landscape claims that drive
build-or-import decisions, reduces the proposed ten-item ownership list,
and records two corrections to earlier Core documents that the landscape
review falsifies.

---

## 1. Citation verification

The landscape review is the input to build-or-import decisions, so its
load-bearing claims were checked rather than assumed. Seven were fetched
directly.

| Claim | Verified | Note |
|---|---|---|
| Parallax, cognitive-executive separation, 98.9% of 280 attacks | **yes** | arXiv 2604.12986, Fokou, Apr 2026 |
| Shepherd, Git-like reversible traces, 5x faster than docker commit | **yes** | arXiv 2605.10913, Yu et al. (Stanford) |
| lambda-RLM, typed lambda-calculus runtime, 4.1x latency | **yes** | arXiv 2603.20105 — actual title is *The Y-Combinator for LLMs* |
| Microsoft Agent Governance Toolkit | **yes** | broader than described: 5 languages, 992 conformance tests |
| Unfireable Safety Kernel | **yes** | arXiv 2606.26057 — 704/704 blocked over 1,000 self-modifications |
| Self-Harness, weakness mining to regression-gated harness change | **yes** | arXiv 2606.09498, Terminal-Bench 2.0 |
| "Gated Semantic Quality-Diversity" | **partly** | see below |

**A distinction that matters for how much this verification buys us.**
For *novelty* claims, being wrong about a paper is safe in one
direction: if a cited system turns out not to exist, Core's uniqueness
only increases. But for *build-or-import* decisions the error is not
safe — deciding to delegate replay to Shepherd is worthless if Shepherd
is not real. Verification was therefore concentrated on the papers that
change what we build, not the ones that change what we claim.

### 1.1 The one discrepancy, and it is in the load-bearing place

arXiv 2607.13683 is **HarnessBank: Semantic Gene-Bank Search with Gated
Verification for Agent-Harness Self-Evolution** (Luo et al.). It is not
titled "Gated Semantic Quality-Diversity", and two specifics differ:

```text
attributed   deterministic code owns sampling, measurement,
             significance testing and sealed evaluation
abstract     a task agent paired with a separate evolver agent;
             a gene bank; a gated screening mechanism
             -- the sampling / significance / sealed-evaluation
                split is not stated

attributed   +9 to +15.5 percentage points, seven domains
abstract     5.1% to 15.4%, seven agent benchmarks
```

The stronger split may well appear in the body; only the abstract was
checked. But this is the paper Pillar 4 leans on hardest — "maps almost
exactly onto what Core should do" — and Pillar 4 is the pillar you want
to push hardest. Resting the load-bearing pillar on the one
misattributed citation is the Continuum failure mode in miniature. Read
the body before Pillar 4 is designed, or drop the citation and justify
deterministic crediting from our own four independent-review results,
which are first-hand evidence and do not need a paper behind them.

### 1.2 A consequence the synthesis missed

HarnessBank reports that improvements come from a *model-specific*
self-evolving process **rather than a universally optimal harness**.
Self-Harness likewise reports per-model weakness mining across three
model families.

Core intends cross-provider governed delivery. If learned harness
improvements are model-specific, then Core's learning plane does not
produce *a* better harness — it produces one per model, and an
improvement validated on one model **must not** promote for another.

That is cheap now and expensive later. It means:

- model identity is part of the episode key, not a payload field
- the promotion gate is per-model; a cross-model promotion needs its own
  regression evidence
- "Core got better" is never a global statement

---

## 2. The correction I owe on the effect boundary

Section F of the gap analysis says the typed gate is "the single
clearest thing Core can do differently from generic agent frameworks."

**That is now false and I am retracting it.** Parallax is
cognitive-executive separation as its central thesis. The Unfireable
Safety Kernel is the same boundary with process separation, fail-closed
semantics and signed external evidence. Both predate Core's version.

The worse problem is not novelty. It is that **the mechanism does not
work in Python.**

```python
executor.execute(proposal)   # gap analysis claims: must not type-check
```

It does not type-check *under mypy*. At runtime, Python has no private
constructors. Every one of these mints an `AuthorizedAction` with no
gate involvement:

```text
AuthorizedAction.__new__(AuthorizedAction)
object.__new__(AuthorizedAction)
copy.copy(some_authorized_action)
dataclasses.replace(other, action=whatever)
pickle round-trip via __reduce__
```

Static types are a lint. The gap analysis presented a lint as a
structural guarantee, which is precisely the "announced fix that is
structurally inert" pattern that independent review caught twice in
Continuum. Compare what the prior art actually uses: Parallax runs an
independent validator; the Unfireable Kernel uses process separation,
Rust, and an SMT proof. Core's proposed enforcement is strictly the
weakest of the three.

### 2.1 Graduated enforcement instead

Three tiers, adopted in order of what Core actually governs:

```text
tier 1  static type      mypy, ergonomics, honest signature
                         stops: mistakes
tier 2  capability token HMAC over canonical(proposal, decision),
                         key held by the gate; executor verifies
                         stops: model-authored bypass code
tier 3  process boundary gate out-of-process, signed decisions
                         stops: in-process compromise
```

Tier 2 is roughly twenty lines of stdlib `hmac` and is what makes the
invariant real rather than advisory. A model that writes code cannot
mint a token without locating the key, which is a materially different
proposition from calling a constructor.

Its ceiling should be stated plainly and not talked around: **the key is
in-process, so in-process code can reach it.** Tier 2 defeats accident
and model-authored bypass, not a determined in-process attacker. Tier 3
is the honest answer for anything genuinely destructive, and the
Unfireable Kernel is the design to copy when Core gets there. Core
should ship tier 1 + tier 2 and document tier 3 as the upgrade path
gated on Core governing its first irreversible effect.

## 3. The correction I owe on the state model — held, not re-litigated

Section 10 of the landscape review restates the state model as four
orthogonal dimensions:

```text
epistemic status / decision status / authority status / validity
```

This is the form corrected in gap analysis section E, and the correction
stands. `epistemic` and `decision` are not orthogonal — an item cannot
be both `OBSERVED` and `DECIDED`; those are alternatives, not
coordinates. Modelling them as two nullable fields makes illegal states
representable, which is the mirror of the collapsing error rather than a
fix for it. The committed form is one discriminator plus two genuinely
orthogonal axes:

```yaml
kind:       OBSERVATION | INTERPRETATION | DECISION   # exactly one
authority:  INFORMATIONAL | AUTHORIZED                # orthogonal
validity:   ACTIVE | STALE | SUPERSEDED               # orthogonal
```

Noted only because the four-way form reappeared; no new argument is
offered and none is needed.

---

## 4. The reduction

The proposed list of ten, each classified by whether Core must **own**
new code and semantics, has already **built** it in v1, or should
**import** it.

| # | Item | Verdict |
|---|---|---|
| 1 | Software-delivery lifecycle semantics | **BUILT** — v1 has it, migrate |
| 2 | Typed project truth | **DESIGNED** — section E, small new code |
| 3 | Proposal → authorization → execution | **OWN** — but redesign per §2 |
| 4 | Evidence-carrying closure | **BUILT** — extend from checks to effects |
| 5 | Recursive lineage + authority + budgets | **CUT** — see §5 |
| 6 | Verified-episode schema | **OWN** — highest value item |
| 7 | Causal / mutation verification of Core | **BUILT** — M1–M8 registry exists |
| 8 | Learning eligibility | **OWN** — this is the thesis |
| 9 | Bounded editable harness surface | **OWN** — a manifest and a check |
| 10 | Regression / adversarial promotion gate | **OWN** the composition only |

Three are already built. One is designed. One is cut. Five remain — and
those five are three subsystems, because 4+6 are one artifact and 9+10
are one gate.

```text
K  AUTHORITY KERNEL      what may act, with what scope,
                         and what the learning plane may never touch
                         = items 3 + 9

E  EPISODE RECORD        the verified-episode schema, and closure
                         carrying evidence for effects as well as checks
                         = items 6 + 4

P  PROMOTION             which episodes are eligible to teach, and who
                         certifies an improvement -- never the improver
                         = items 8 + 10
```

**K, E, P.** Everything else is migrated from v1 or imported.

The dependency runs E → P → K in build order but K → P → E in trust
order, and the trust direction is the invariant: P cannot weaken K, and
E cannot cause an effect.

### 4.1 What "import" means concretely

```text
orchestration / checkpointing      LangGraph
recursion, if ever needed          lambda-RLM (typed combinators)
trace fork / replay semantics      Shepherd's model, before inventing ours
generic policy / identity / sandbox Microsoft AGT
process-isolated kernel, tier 3    Unfireable Safety Kernel
weakness mining from traces        Self-Harness
tool transport                     MCP 2026-07-28
```

---

## 5. The cut: Core v1 is depth-1

Recursive autonomy should leave the critical path entirely.

The research question is whether a delivery agent can improve its
execution policy from verified experience while structurally unable to
self-authorize or self-certify. **Recursion appears nowhere in that
sentence.** It is a capability for decomposition and long context, not a
component of the claim.

It is also the worst thing on the list to build:

```text
most crowded          RLM, lambda-RLM, RAH all occupy it
contested benefit     a reproduction finds deeper recursion can hurt
                      accuracy while raising time and cost
highest cost          a recursive executor with budget accounting
                      across a tree with retries and partial failure
orthogonal            the thesis holds identically at depth 1
```

Building it would spend the most implementation effort on the pillar
with the most prior art and the least connection to what Core is trying
to prove.

**Keep the fields, not the machinery.** The episode schema reserves
`parent_episode_id` and `authority_scope` from day one, so depth-N is a
later addition rather than a migration. Authority narrowing is a set
intersection, not a subsystem, and can be written when there is a second
level to narrow to.

Pillar 3 is demoted from a pillar to two schema fields.

---

## 6. The other thing to drop: the dataset moat

Section 19 of the landscape review proposes that the strongest moat is
the trajectory dataset after 10,000 governed tasks. Three problems:

```text
volume        at realistic single-operator throughput, 10,000 governed
              tasks is a multi-year horizon, not a moat we can reach
distribution  single-user data is single-distribution -- it teaches
              Core to be good at these repositories, which is a fine
              product outcome and not a general capability
transfer      HarnessBank's own finding is that gains are
              model-specific; the dataset does not cleanly transfer
              across models even in principle
```

The episode schema survives this unchanged, because its justification
was never the moat — it is the substrate the eligibility predicate
operates on. Item 6 stays at the top of the list. The *moat* framing
should go, because it sets a bar Core will not clear and invites a claim
we cannot support.

## 7. Language discipline

**"Proof-carrying" is retired. "Evidence-carrying" only.**

Proof-carrying code is a real formal artifact (Necula, 1997) shipping
machine-checkable proofs against a stated safety policy. Core will ship
evidence — hashes, check outcomes, approvals, traces. Borrowing the
phrase invites exactly the comparison Core loses, in front of exactly
the reviewers whose opinion matters. The landscape review already
hedges this; making it a rule removes the temptation.

The claim split proposed in section 22 is adopted as written, with one
tightening — the engineering hypothesis is the falsifiable one and needs
a defined failure condition before any learning code is written:

```text
product      Core makes autonomous software delivery governable,
             verifiable and improvable
engineering  verified execution traces can drive useful harness
             adaptation without the adaptation process controlling
             its own promotion criteria
research     governed self-improvement can improve delivery outcomes
             over a fixed harness while preserving invariant
             authorization and verification guarantees
```

Only the third requires serious empirical proof. The second requires an
experiment. The first is a product statement and should never be cited
as though it were either.

---

## 8. What this means for the migration sequence

The ordering invariant from ADR-001 is unchanged: the gate exists before
the executor, and the executor exists before the model can propose.
Three amendments follow from the above.

```text
+ tier-2 capability token lands WITH the gate, not after it
  -- an advisory gate is not a gate

- recursive executor removed from the sequence entirely
  + parent_episode_id / authority_scope reserved in the schema

+ episode schema moves EARLIER, ahead of the first executor
  -- it is the record the executor must write into, and retrofitting
     a schema onto an existing executor is how fields become optional
```

Everything downstream of P — the learning plane — remains out of scope
until K and E have shipped and been independently reviewed.

## 9. Open question, surfaced rather than decided

Tier 2 assumes the gate can hold a key the rest of the process should
not use. Within one Python process that is a convention about who
imports what, and Core's own learning plane is eventually code that
Core may modify.

The question is therefore not "can the model forge a token" but **"can
the learning plane's output ever end up in the same process as the
key?"** If yes, tier 2's guarantee degrades to tier 1 the moment the
first learned harness change is loaded. That may force tier 3 earlier
than the "first irreversible effect" trigger proposed in §2.1.

Not resolved here. It should be resolved before the learning plane is
designed, not after.
