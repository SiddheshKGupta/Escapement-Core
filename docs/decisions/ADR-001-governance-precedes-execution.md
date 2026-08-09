# ADR-001 — Governance precedes execution; execution precedes autonomy

**Status:** Accepted. Permanent repository invariant.
**Date:** 2026-08-09
**Applies to:** every commit in Escapement Core.

## Context

> **CORRECTED 2026-08-09 — the context below was false. The decision
> stands; its justification does not.** See
> [EFFECT_SURFACE_AUDIT.md](../core/EFFECT_SURFACE_AUDIT.md). The grep
> covered 3 patterns across 2 of 14 files; widened, it returns 47
> matches in 7 files, including `run_check.py --shell`, the repository's
> one `shell=True` site, which executes a caller-supplied string whose
> only safeguard is the word "trusted" in its help text. Core did not
> start from a clean slate — it inherited an ungoverned path in the
> clone. The corrected position: **v1 adds no execution capability
> beyond what its host already grants the model**, and its runner
> records effects rather than authorising them. Core's differentiator is
> therefore not that governance came first, but that Core is the point
> at which routing an effect through the recorder stops being optional.
> The ordering invariant is *more* necessary under the corrected facts,
> not less.

Escapement v1 has **no model-to-effect path**. Verified at the commit
this repository was derived from:

```bash
grep -rniE "subprocess|os\.system|exec\(" scripts/agent_runtime.py \
                                          scripts/capability_router.py
# no matches
```

v1 governs by composing instructions that a host then follows.
Subprocess execution exists only in operator-invoked CLI paths.

This is an unusual and valuable starting position. Most agent frameworks
grow an execution path first and retrofit safety around it, which means
there is always a period — often permanent — in which an ungoverned path
exists and every subsequent control is a patch over it.

Core does not have to inherit that. It can establish governance *before*
autonomous execution exists at all.

That advantage is fragile. It survives only if the ordering is treated
as an invariant rather than a preference, because the pressure to "just
wire the model up and add gates after" will be constant and will always
appear reasonable in the moment.

## Decision

```text
Effect Gate   exists before
Executor      exists before
Model         can propose executable actions
```

**No commit may introduce an ungoverned model-controlled execution path.**

Concretely:

1. The effect gate and its adversarial tests land before any executor.
2. An executor accepts `AuthorizedAction` only. `AuthorizedAction` has no
   public constructor and is producible solely by the gate, after effect
   classification, policy check, authority check, and any required
   approval.
3. A model adapter may produce `ActionProposal` and nothing else. It has
   no path to an executor that does not pass through the gate.
4. Bypass is a **type error**, not a code-review finding.

## Consequences

**Accepted cost.** Core will have a governed execution path that nothing
uses for several milestones, and a model adapter arrives late. That is
the point. Building the gate against a real executor that already works
is how the gate ends up shaped around what the executor happens to do.

**Required tests, before any executor ships:**

- reaching an executor without the gate **fails the build**;
- constructing `AuthorizedAction` outside the gate fails;
- a `DESTRUCTIVE` proposal without approval is denied, and the denial is
  recorded in the event trace.

**Ordering is checkable.** Migration steps are numbered; the gate is
step 6–7, the executor 14–16, the model adapter 17–18. A commit that
introduces an executor before the gate, or a model path before the
executor's bypass validation, violates this ADR regardless of how the
code reads.

## Why this is stated as an ADR rather than a guideline

The Continuum research programme produced four documented cases in which
self-authored work passed its author's own verification and independent
review found real defects — including two fixes announced as complete
that were structurally inert, and a checker that passed the exact defect
it was written to catch.

The lesson carried forward is not "be more careful". It is that
invariants must be **structural and externally checkable**, because the
author's judgement in the moment is not reliable evidence. An ordering
rule that lives only in a plan is a preference. One that fails the build
is an invariant.

## Related

- `docs/core/CORE_READINESS_GAP_ANALYSIS.md` — sections E, F, L
- `ESCAPEMENT_CORE_MASTER_HANDOFF.md` §26, §38
