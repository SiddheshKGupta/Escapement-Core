# ADR-002 — What Escapement Core owns

```text
STATUS    Accepted, as amended 2026-08-09
DATE      2026-08-09
CONTEXT   docs/core/ARCHITECTURE_REDUCTION.md
SUPPORTS  ADR-001 (governance precedes execution)
AMENDED   D1 restated (six responsibilities, three of them inherited)
          D2 superseded by ADR-003 (process boundary, not HMAC)
          D3 ruled: DEFERRED, not cut
```

## Context

A landscape review of 2025 – August 2026 established that every
individual edge of Core's proposed architecture already exists:
cognitive-executive separation (Parallax), governance breadth (Microsoft
AGT), external authorization kernels (Unfireable Safety Kernel),
branchable execution traces (Shepherd), typed recursion (lambda-RLM),
harness self-improvement (Self-Harness, HarnessBank). Seven of these
were verified directly rather than taken on trust.

No single edge can be Core's contribution. The reduction asks what code
and semantics remain after everything importable is imported.

## Decisions

**D1 — Core has six responsibilities. Three are new; three are already
earned.** *(restated 2026-08-09)*

The original wording — "Core owns three subsystems" — was a reduction of
what must be *invented*, and reads as a description of what Core *is*.
It is not: it writes v1's earned semantics out of Core's own identity.

```text
INHERITED FOUNDATION -- already earned by v1, migrated not reinvented

L  DELIVERY LIFECYCLE   Specify -> Route -> Execute -> Verify -> Persist
V  VERIFICATION/CLOSURE evidence-backed completion; assertion is not PASS
T  TEST DISCIPLINE      causal sensitivity, mutation, independent review

NEW CORE SEMANTICS -- the three differentiating responsibilities

K  CONSTITUTIONAL KERNEL authority, effect governance, promotion boundary
E  EPISODE/EVIDENCE      what happened, why, cost, result, verification
P  LEARNING & PROMOTION  what may teach Core, what may change, what ships
```

Orchestration, policy breadth, identity, sandboxing, replay semantics
and transport are **imported**.

> **AMENDED on independent review.** Two errors here.
>
> **"Trust flows K → P → E and never back" is deleted.** `P` consumes
> `E`, so evidence necessarily flows `E → P`. One word was carrying
> three distinct flows — exactly the collapse Continuum was supposed to
> have taught us:
>
> ```text
> AUTHORITY  operator/policy -> K -> {worker, promotion}
> EVIDENCE   worker -> events -> E -> P
> CHANGE     P proposes -> K/promotion gate -> mutable worker surface
> ```
>
> **K/E/P is not irreducible, and `P` is an extension, not a pillar.**
> Remove `P` and `L+V+T+K+E` is still a coherent product: a governed
> delivery runtime producing verified execution episodes. `P` depends on
> `E`; `E` does not depend on `P`; `K` does not require `P`.
>
> ```text
> CORE BASE               L + V + T + K + E
> RESEARCH EXTENSION      P
> ```
>
> One thing this sharpens rather than softens: `K` is *not* optional.
> `L+V+T+E` without `K` is approximately v1 and AGET, which exist;
> `E+P` without `K` is approximately Self-Harness, which exists. Every
> configuration lacking `K` is already occupied by prior art. So `K` is
> necessary for differentiation though not sufficient — and Core v1 can
> now justify its existence without the self-improvement research
> succeeding, which is a strictly better position.

**D2 — The effect boundary is enforced by capability token, not by type
alone.** *(SUPERSEDED by [ADR-003](ADR-003-kernel-worker-separation.md) —
the process boundary is the trust boundary, and it is designed before
the first executor rather than deferred behind it. The retraction below
stands; the remedy does not go far enough.)*

Retracts the gap analysis claim that a Python type makes bypass
structurally impossible. It does not: Python has no private
constructors, and `__new__`, `copy`, `dataclasses.replace` and pickle
all reach around one. Static typing is a lint, and a lint presented as a
guarantee is the same defect class independent review caught twice in
Continuum.

Enforcement is graduated: static type for ergonomics, HMAC capability
token for runtime, process separation as the documented upgrade path.
The token lands **with** the gate — an advisory gate is not a gate.

**D3 — Recursive execution is DEFERRED and architecturally reserved, not
cut.** *(ruled 2026-08-09; amends the proposal below)*

The proposal was to cut recursion outright. The ruling accepts the
deferral and rejects the removal, and the distinction is real: "reduced
to two schema fields" would have quietly decided that recursion is not
part of Core's long-term architecture, which is a larger claim than the
evidence supports and not one the first hypothesis requires either way.

Recursion is absent from the first research question, occupies the most
crowded prior art, has contested benefit in a published reproduction,
and carries the highest build cost on the list — so it is out of the
first autonomous milestone. But it is not out of Core.

```text
ExecutionStrategy = DIRECT | RLM | RECURSIVE_HARNESS

RLM != Core          it is a strategy Core may route to
RLM != learning      the Learning Plane decides when it has helped
```

Recursion re-enters as a **routed execution strategy that must beat the
direct baseline on verified outcomes**, per task class, at a promotion
scope no wider than the evidence supports. That is a stronger position
than baking it into the kernel: it makes recursion falsifiable rather
than assumed, and it is the same admission discipline Continuum earned.

Sequence: direct Core → governed execution → verified episodes →
learning and promotion → direct baseline established → recursion
admitted as a strategy → direct vs recursive comparison.

Reserved from the first episode ever recorded, never backfilled:
`parent_episode_id`, `authority_scope`, and the delegated-authority and
resource fields.

**D4 — Learning is per-model.**

HarnessBank reports gains are model-specific rather than a universally
optimal harness. Model identity is part of the episode key, the
promotion gate is per-model, and an improvement validated on one model
does not promote for another without its own evidence.

**D5 — "Evidence-carrying", never "proof-carrying".**

Proof-carrying code is a real formal artifact with machine-checkable
proofs. Core ships evidence, not proofs. The dataset-moat framing is
also dropped: single-operator volume will not reach it, single-user data
is single-distribution, and per D4 it does not transfer across models.

## Consequences

The episode schema moves ahead of the first executor, because
retrofitting a record onto a running executor is how required fields
become optional. The learning plane stays out of scope until K and E
have shipped and been independently reviewed.

One question is deliberately left open in the reduction document: if the
learning plane's output can ever load into the same process as the gate
key, tier-2 enforcement degrades to tier-1 at that moment. That may
force process separation earlier than planned. It must be resolved
before the learning plane is designed.

## Alternatives rejected

**Compete with Microsoft AGT on generic governance.** Five languages,
992 conformance tests, OWASP Agentic Top 10 coverage. Specialisation to
software delivery is the position; breadth is not available.

**Build Core as a Darwin Gödel Machine.** Open-ended self-rewriting is
the thing D1's kernel exists to prevent. DGM is retained as the
comparison for what Core deliberately constrains.

**Ship the typed boundary alone and add runtime enforcement later.**
Rejected under ADR-001: an unenforced gate at the moment the first
executor lands is precisely the ungoverned window the ordering invariant
exists to prevent.
