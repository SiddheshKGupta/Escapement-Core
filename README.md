# Escapement Core

Governed execution runtime for AI coding agents. Design phase — **no Core implementation exists yet.**

```text
Status        design frozen, awaiting EXPERIMENT_CORE_000
Implemented   nothing beyond the inherited v1 runtime
Licence       source-available (v1 is Apache-2.0 — see Licence)
```

## What this is

v1 governs how an agent *should* work. Core is an attempt to make Escapement part of the mandatory path through which consequential work happens: the model proposes, Core authorizes, effects produce evidence.

Whether that requires Core to own an executor is **an open question**, not an assumption. `EXPERIMENT_CORE_000` tests it against the strongest existing composition — v1 semantics + a host sandbox + Microsoft AGT + CI — before any executor is built.

## Why it is not built yet

Four rounds of independent review cut the scope each time:

```text
10 ownership items      -> 3 new subsystems, 3 inherited from v1
typed effect boundary   -> retracted; Python has no private constructors
in-process HMAC         -> superseded by kernel/worker process separation
recursion (RLM/RAH)     -> deferred; must beat a direct baseline first
"v1 has no effect path" -> false; 47 effect primitives in 7 files
```

The last one falsified the founding claim — see [`EFFECT_SURFACE_AUDIT.md`](docs/core/EFFECT_SURFACE_AUDIT.md).

## Scope

```text
INHERITED   L  delivery lifecycle
            V  verification and evidence-backed closure
            T  test discipline: causal sensitivity, mutation

NEW         K  constitutional kernel: authority, effect gate, promotion
            E  episode record: verified execution episodes
            P  learning and promotion: what may teach Core

IMPORTED    LangGraph, Microsoft AGT, Shepherd, MCP, lambda-RLM
```

`P` is an extension, not a pillar. `L+V+T+K+E` is a coherent product without it.

## Invariant

> Governance precedes execution; execution precedes autonomy.

```text
Effect Gate   before   Executor   before   model-proposed actions
```

No commit may introduce an ungoverned model-controlled execution path. [ADR-001](docs/decisions/ADR-001-governance-precedes-execution.md).

## Documents

| Document | Contents |
|---|---|
| [Gap analysis](docs/core/CORE_READINESS_GAP_ANALYSIS.md) | v1 reuse map, state model, migration order |
| [Architecture reduction](docs/core/ARCHITECTURE_REDUCTION.md) | landscape review, 8 citations verified |
| [Effect surface audit](docs/core/EFFECT_SURFACE_AUDIT.md) | what v1 can actually execute |
| [Threat model v0.1](docs/core/CORE_THREAT_MODEL_v0.1.md) | 5 trust classes, 1 adversary, P1-P10 |
| [Episode schema v0.1](docs/core/CORE_EPISODE_SCHEMA_v0.1.md) | episode record, learning eligibility |
| [Experiment 000](docs/core/EXPERIMENT_CORE_000_PREREGISTRATION.md) | the gate on building any executor |

ADRs in [`docs/decisions/`](docs/decisions/): `001` ordering, `002` ownership, `003` kernel/worker, `004` the pause, `005` the freeze.

## Lineage

Cloned from [Escapement v1](https://github.com/SiddheshKGupta/Escapement) with full history. **No GitHub fork relationship** — PRs must not default upstream, and the repository must stay freely private-able.

v1 is the stable line and is not deprecated. [Continuum](https://github.com/SiddheshKGupta/Escapement-Continuum) is a frozen research programme; Core inherits its lessons, not its machinery.

```text
v1         proves the harness
Core       industrialises the trust boundary
Continuum  researches what Core still cannot solve
```

## Inherited v1 runtime

```text
Native skills:                35
Capability strengths:         58
Governed external resources:  67
Unit tests:                   189
Routing evaluations:          122
Repository files:             296
```

```bash
python scripts/escapement.py doctor --root .
python scripts/eval_harness.py run
python -m unittest discover -s tests -p "test_*.py"
```

## Licence

Source-available, not OSI approved. See [`LICENSE.md`](LICENSE.md).

**v1 is Apache-2.0.** Files inherited from v1 are therefore also obtainable under Apache-2.0 from that repository, so this licence meaningfully covers only Core-specific work. Core may relicense once Experiment 000 has run.
