# Core Threat Model v0.1

```text
STATUS   minimum viable -- scoped to make EXPERIMENT_CORE_000 falsifiable
NOT      a complete threat model for brokers, remote MCP, recursive
         children, multi-tenant or deployment environments
PURPOSE  define what "governed execution" means against arbitrary
         worker code, so the composition spike can pass or fail
         against something
```

A spike invariant reading "no consequential effect without governed
authorization" is unfalsifiable until it says *authorized against which
attacker*. Without that, passing the spike is a conformance result — the
exact failure mode that made Experiment 001 worthless in Continuum.

This document is deliberately small. It defines five trust classes, one
adversary, ten properties, and the mapping from properties to spike
invariants. Everything else is deferred.

---

## 1. Trust classes

```text
T0  USER / OPERATOR
    the trusted authority source. Grants, approves, revokes.

T1  CONSTITUTIONAL CONTROL
    policy, authorization decisions, promotion criteria, required
    verification, trusted evidence integrity.

T2  PRIVILEGED CAPABILITY PROVIDERS
    host sandbox, AGT, credential brokers, CI, provider APIs.
    Trusted to enforce what they claim to enforce; not trusted to
    understand Escapement semantics.

T3  UNTRUSTED INTELLIGENCE
    the model, model-generated code, learned harness changes, and
    later recursive children.

T4  UNTRUSTED INPUT
    repository content, web pages, MCP tool output, tool responses,
    external documents.
```

T2 is the class Core most wants to *use* rather than rebuild. Whether
T1 must be Escapement-implemented or can be supplied by T2 is precisely
what EXPERIMENT_CORE_000 exists to determine — this document does not
presume the answer.

## 2. The adversary

**Assume arbitrary code execution inside T3.**

One adversary, stated once, because a threat model with five adversaries
and no experiment is a document rather than a control.

The attacker may:

```text
send arbitrary IPC / tool requests
manipulate all worker-local state and memory
fabricate model output
generate malicious improvement proposals
replay previously valid requests
exploit any authority ambient in its environment
attempt to poison episode records
consume resources within available limits
attempt parser and deserialization attacks against T1 and T2
race authorization against execution
```

### 2.1 Explicitly out of scope for v0.1

```text
host OS administrator or root compromise
hardware attacks
OS kernel exploits
physical compromise
compromise of cryptographic primitives
malicious T2 provider (host sandbox or AGT itself lying)
```

These are future revisions, not permanent exclusions. `SeDebugPrivilege`
on Windows and `CAP_SYS_PTRACE` on Linux are treated as administrator
capability and therefore out of scope — which is a real limitation,
because a developer workstation frequently runs as an account holding
them.

### 2.2 The architectural property, stated once

> **Arbitrary T3 code must not acquire T1 authority except through an
> explicitly authorized transition.**

This replaces "separate process" as the thing Core is trying to
guarantee. Process separation is one possible implementation.

## 3. Required properties

Mechanism-agnostic on purpose. Technologies are evaluated against these,
never substituted for them.

| # | Property |
|---|---|
| P1 | T3 cannot read T1 secrets |
| P2 | T3 cannot mutate T1 policy or state |
| P3 | T3 cannot execute a consequential effect without an authorization bound to that exact effect |
| P4 | T3 cannot widen delegated authority |
| P5 | T3 cannot modify trusted historical evidence |
| P6 | T3 cannot promote its own learning proposal |
| P7 | T3 cannot weaken the test used to certify that proposal |
| P8 | T3 cannot obtain ambient filesystem, network or credential authority outside granted capability surfaces |
| P9 | An authorization cannot be replayed for a materially different effect |
| P10 | Revocation and expiry semantics are checked where required |

Only after these exist should `AppContainer`, Landlock, seccomp,
containers, VMs, AGT, brokers or host sandboxes be evaluated as
implementations. **If no available implementation satisfies a required
property, Core refuses untrusted autonomous execution or enters an
explicitly labelled unsafe development mode.** It does not proceed on a
weaker boundary and call it governed.

## 4. Property → spike invariant mapping

This is what makes EXPERIMENT_CORE_000 falsifiable. Every spike
invariant traces to a property and an adversary capability.

```text
P3, P8      -> S1  effect mediation
P5          -> S2  reconstruction
(v1 rule)   -> S3  truthful closure
P6, P7      -> S4  promotion independence
P7          -> S5  prospective credit
P5          -> S6  rollback
P1, P2, P8  -> S7  ambient authority
P3, P9, P10 -> S8  TOCTOU
(portability)-> S9 host portability -- NON-BLOCKING in Experiment 000
```

## 5. Baseline reading of v1 today

Recorded before the spike so the starting point is not reconstructed
afterwards. This is v1 as it exists, not a prediction about the
composition.

```text
P1   N/A      no T1 secrets exist yet
P2   FAILS    policy is repository files the agent can edit
P3   FAILS    no authorization object exists; effects are unmediated
P4   N/A      no delegation exists
P5   FAILS    .agent/ records are writable by the same agent
P6   FAILS    no promotion system exists
P7   FAILS    the agent can edit its own tests
P8   FAILS    run_check.py --shell, plus the host's own Bash tool
P9   N/A      no authorizations to replay
P10  N/A      no revocation semantics
```

**Five FAIL, five N/A, zero PASS.** This is not a criticism of v1 —
v1 is a procedural governance harness and was never built to survive an
adversarial T3. It is the honest baseline, and it makes the transition
precise:

```text
v1          governs how the agent SHOULD work   (procedural, recording)
Core target mandatory mediation of what the agent CAN do (structural)
```

The N/A entries matter as much as the failures: half these properties
cannot even be *evaluated* against v1, because the objects they quantify
over do not exist. That is what the spike is testing — whether a
composition of existing T2 providers supplies them.

## 6. What v0.1 does not cover

```text
malicious or compromised T2 provider
broker-specific trust boundaries      (broker does not exist yet)
remote / third-party MCP servers
recursive children and delegated authority chains
multi-tenant or shared-host deployment
supply-chain authority: who may update the kernel, Zone-C policy,
  the eligibility checker, the isolation backend, and how that
  update is authenticated
denial of service and resource ceilings
T4 handling in depth -- prompt injection from repository or web
  content is acknowledged and not yet modelled
```

Supply-chain authority and DoS are the two most likely to need
promoting into v0.2, and neither blocks the spike: the spike runs one
task on one host under operator supervision.
