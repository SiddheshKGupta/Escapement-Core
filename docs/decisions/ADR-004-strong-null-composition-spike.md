# ADR-004 — Pause the executor; run the strong-null composition spike

```text
STATUS    Accepted
DATE      2026-08-09
EFFECT    no Core executor, kernel/worker IPC, broker or MCP surface
          is authorized until this spike runs and is scored
BLOCKS    ADR-003 implementation (already PROPOSED)
```

## Context

Independent review raised a null that Core has not defeated:

```text
Escapement v1 delivery semantics
  + Claude Code / Codex OS-enforced sandbox and approval policy
  + Microsoft AGT governance, capability model, tamper-evident audit
  + ordinary CI and independent verification
  + an external Self-Harness-style improvement loop
  + optional Shepherd replay
```

Against which: *what exact capability requires Core to own a new
executor and a new constitutional kernel?*

This is the same question that ended Continuum Phase I, arriving at the
same point in the lifecycle — after an architecture felt settled and
before it was built. There, specifying the strongest fair comparator
removed every candidate distinction in turn until nothing was left to
test. The 180-run experiment was cancelled before execution. That was
cheap. Building a cross-platform security boundary, an IPC protocol, a
privileged broker, an event ledger and a promotion system before asking
the same question would not be.

## Decision

**No Core executor is authorized until a composition spike has been run
and scored against seven invariants.**

Build one bounded path using only existing components:

```text
v1 delivery semantics
      -> Claude Code or Codex as executor
      -> host governance + AGT
      -> v1 evidence closure
      -> normalized episode record
      -> offline improvement proposal
      -> external promotion gate
```

One real MATERIAL-tier task. Then score:

```text
I1  no consequential effect without governed authorization
I2  all effects reconstructible from event/evidence records
I3  model assertion alone cannot produce PASS
I4  learning proposer cannot change promotion criteria
I5  candidate improvement evaluated against a frozen baseline
I6  improvement can be rolled back
I7  same episode semantics portable to a second host
```

**If the composition satisfies all seven, the executor is not built.**
Core shrinks to whatever semantic layer remains missing. That is a win,
not a defeat — it is the cheapest possible discovery that most of Core
is unnecessary.

**If it fails, the failing invariant is the justification for Core code**
— and only for the code that invariant requires.

## Preregistered predictions

Recorded **before** the spike runs, because Continuum's decision journal
held twelve entries of which eight were retrospective and zero were
resolved predictions. A prediction written afterwards is not evidence.

```text
I1  PASS expected     host permission systems plus AGT policy plausibly
                      cover this for the tool surface they mediate
I2  FAIL predicted    hosts expose transcripts, not complete typed
                      effect records; reconstructing every effect from
                      what Claude Code or Codex emits is the weak point
I3  PASS expected     v1 already enforces this and it is inherited
I4  FAIL predicted    nothing in the stack governs promotion of harness
                      changes; AGT governs agent actions, not the
                      lifecycle of the harness itself
I5  PASS expected     ordinary CI against a frozen baseline
I6  PASS expected     version control
I7  UNSCORED          requires a second host; out of scope for v0
```

Predicted result: **five pass, two fail (I2, I4)**. If that holds, Core's
justified surface is the episode record and the promotion boundary —
`E` and the promotion half of `K` — and *not* an executor, since I1
passing means the host's execution path is adequately governed.

**A prediction of two failures is itself a hypothesis and may be wrong
in the direction that costs us more.** If I2 and I4 also pass, Core
reduces to a normalisation layer over existing hosts and should be built
as one. That outcome must be recorded as readily as the convenient one.

## Consequences

`CORE_EPISODE_SCHEMA_v0.1.md` survives regardless and is the right next
artifact: the spike needs a normalized episode record to score I2 and
I7, so the schema is the spike's instrument rather than a bet on the
executor being built.

`CORE_THREAT_MODEL_v0.1.md` is deferred, not cancelled. It is required
before ADR-003 can leave `PROPOSED`, and ADR-003 cannot be implemented
before this spike anyway.

## Alternatives rejected

**Build the executor and run the spike in parallel.** The spike's only
value is that it can stop work. Running it alongside the work it might
stop converts it into documentation.

**Skip the spike; the reasoning is clear enough.** The reasoning was
also clear before the effect surface audit, which found the founding
claim false. Reasoning that has been wrong once at this exact level of
confidence does not get to substitute for a test the second time.
