# ADR-002 — What Escapement Core owns

```text
STATUS    Proposed -- one decision needs your ruling (D3)
DATE      2026-08-09
CONTEXT   docs/core/ARCHITECTURE_REDUCTION.md
SUPPORTS  ADR-001 (governance precedes execution)
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

**D1 — Core owns three subsystems, not ten items.**

```text
K  AUTHORITY KERNEL   what may act, with what scope, and what the
                      learning plane may never modify
E  EPISODE RECORD     the verified-episode schema, and closure carrying
                      evidence for effects as well as checks
P  PROMOTION          which episodes may teach, and who certifies an
                      improvement -- never the improver
```

Lifecycle semantics, evidence-carrying closure and mutation discipline
are **inherited from v1**, not invented. Orchestration, policy breadth,
identity, sandboxing, replay semantics and transport are **imported**.

**D2 — The effect boundary is enforced by capability token, not by type
alone.**

Retracts the gap analysis claim that a Python type makes bypass
structurally impossible. It does not: Python has no private
constructors, and `__new__`, `copy`, `dataclasses.replace` and pickle
all reach around one. Static typing is a lint, and a lint presented as a
guarantee is the same defect class independent review caught twice in
Continuum.

Enforcement is graduated: static type for ergonomics, HMAC capability
token for runtime, process separation as the documented upgrade path.
The token lands **with** the gate — an advisory gate is not a gate.

**D3 — Core v1 is depth-1. Recursive autonomy is cut.** *(needs your
ruling)*

Recursion is absent from the research question, occupies the most
crowded prior art, has contested benefit in a published reproduction,
and carries the highest build cost of anything on the list.
`parent_episode_id` and `authority_scope` are reserved in the schema so
depth-N is a later addition rather than a migration.

This reverses part of the proposed Pillar 3, so it is yours to overrule.

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
