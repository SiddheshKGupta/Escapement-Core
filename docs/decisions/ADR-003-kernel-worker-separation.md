# ADR-003 — Trusted kernel, untrusted worker

```text
STATUS    PROPOSED -- downgraded from Accepted 2026-08-09
BLOCKED   ON  a threat model, and one enforceable isolation profile
DATE      2026-08-09
AMENDS    ADR-002 D2 (graduated enforcement)
SUPPORTS  ADR-001 (governance precedes execution)
```

> **Downgraded on independent review.** A trust boundary without an
> adversary model is underspecified, and this ADR was accepted without
> one. It also names a *mechanism* ("process separation") where it
> should name a *property*. Both are corrected in §6 below, and the
> status stays `PROPOSED` until `CORE_THREAT_MODEL_v0.1.md` exists and
> at least one platform isolation profile is shown to satisfy the
> property on paper.

## Context

ADR-002 D2 proposed graduated enforcement — static type, then HMAC
capability token, then process separation "gated on Core governing its
first irreversible effect." The reduction document itself then noted the
flaw: if the learning plane's output can load into the same process as
the gate key, tier-2 degrades to tier-1 the moment the first learned
change loads.

Since model-generated and learned code is a *deliberate* Core
capability, that is not a future edge case. It is the design.

## Decision

**Kernel/worker process separation is designed before the first
executor, not deferred behind it.**

```text
TRUSTED KERNEL                 |  UNTRUSTED WORKER
policy, authority, Effect Gate |  model, generated code, tools,
promotion gate, event integrity|  shell, MCP, learned strategies
credentials                    |
                               |  LEARNING WORKER (later)
                               |  analyses episodes, proposes changes
```

Enforcement roles are reassigned:

```text
static type       ergonomics and mypy. Not security. Unchanged.
capability token  audit and dispatch integrity. Demoted from
                  "the runtime guarantee" to a correlation mechanism.
process boundary  THE trust boundary.
```

The invariant this buys: **Core can modify the worker without giving the
worker the ability to modify the authority that governs it.** Process
separation is the physical expression of ADR-001's ordering rule.

## Consequences

The episode schema becomes the wire format, not merely a record —
everything crossing the boundary must be serialisable and interpretable
by a process that shares no memory with its producer. This is a
constraint on `CORE_EPISODE_SCHEMA_v0.1.md`, and a reason that document
correctly comes first.

## 4. Unresolved problem 1 — process separation alone is not a trust boundary

Two same-user processes are not isolated from each other. The exact
strength is OS- and configuration-dependent, and on this project's
primary platform it is weak:

```text
Windows   a same-user process may OpenProcess a sibling with
          PROCESS_VM_READ and read the key straight out of memory.
          Same-user separation buys approximately nothing.

Linux     with yama ptrace_scope >= 1 a non-ancestor cannot ptrace the
          kernel, and /proc/<pid>/mem enforces the same check. Because
          the kernel spawns the worker, the worker is the child and
          cannot trace its parent. Meaningfully stronger -- but it
          depends on a sysctl the deployment may not set.
```

So "separate process" is necessary and not sufficient. The boundary is
only real with **privilege separation**: kernel and worker as different
OS principals, or the worker confined to a container or sandbox with the
kernel outside it.

Two rules follow, and both are cheaper to adopt now than to retrofit:

```text
the kernel never loads worker-produced code
the kernel treats every byte arriving over IPC as untrusted input
```

The second is easy to state and easy to violate: a kernel that
deserialises a worker-supplied object graph into live Python objects has
readmitted the worker into its own process.

**Not resolved here:** which isolation mechanism Core targets. The
Unfireable Safety Kernel's answer is a separate Rust process with signed
evidence verifiable outside the trust boundary. That is the reference,
but adopting it wholesale is a language decision Core has not made.

## 5. Unresolved problem 2 — credentials and tools are on opposite sides

The boundary as drawn puts `credentials` in the kernel and
`tools / shell / MCP` in the worker. Those conflict. Tools need
credentials to do anything.

Three resolutions, none free:

```text
A  worker receives credentials
   -> defeats the boundary entirely. Rejected.

B  kernel proxies every credentialed call
   -> the boundary holds, but the kernel grows an egress path for
      every protocol Core supports. A large kernel is a large trusted
      computing base, which is the opposite of what a kernel is for.

C  kernel mints short-lived, narrowly-scoped, per-action credentials
   and hands those to the worker
   -> the worker holds a credential it cannot widen or reuse; the
      kernel stays small. Requires the credential's authority to be
      expressible per action, which is true for tokens Core issues
      and often false for a third-party API key.
```

C is the right default with a B fallback for credentials that cannot be
narrowed. It needs deciding before the first executor, because it
determines whether the kernel has an egress stack.

## 6. Corrections from independent review

**The constitutional invariant is a property, not a mechanism.**
"Different PID" is not the invariant, and neither is "container".
Windows checks `OpenProcess` against the target's security descriptor
and `SeDebugPrivilege` can override it; Linux Yama's behaviour depends
on configuration; Landlock's own documentation warns that namespaces are
not fine-grained access control. So the required property is:

> The untrusted worker cannot read or modify kernel memory or state,
> inherit privileged handles, access kernel-held credentials,
> impersonate kernel authority, or reach filesystem and network
> resources except through explicitly granted capability surfaces.

Mechanism becomes a pluggable `IsolationBackend` — AppContainer or
equivalent on Windows, privilege separation plus `no_new_privs`, ptrace
restrictions and seccomp/Landlock on Linux, a VM where the stakes
justify it. **If no available backend satisfies the property, Core
refuses untrusted autonomous execution or enters an explicitly labelled
unsafe development mode.** It does not proceed quietly on a weaker
boundary.

**§5's credential problem has a fourth option, and it is the right
one: a privileged capability broker.** The kernel authorises; a separate
privileged broker holds the broad secret and exposes only a narrow
operation surface; the worker never holds a secret at all.

```text
kernel (authorisation only) -> receipt -> worker -> broker -> provider
                                                    holds the secret
```

This supersedes the A/B/C framing above. Provider-native scoped
credentials are preferred where they exist; the broker covers
third-party keys that cannot be narrowed. Crucially it keeps the
protocol-speaking code *out* of the kernel, so the trusted computing
base stays small — which was the objection to option B. A broker is
itself a new trust class and needs its own boundary in the threat model.

**Time-of-check/time-of-use is unhandled.** An authorisation valid at
issue can be invalid at execution, and `authorize A → modify A →
execute A'` is currently possible. Receipts must bind an exact action
digest, plus expiry, nonce, policy version, and one-shot semantics.

## Alternatives rejected

**Ship in-process HMAC and separate later.** The reason to reject this
is not that HMAC is weak — it is that the separation gets
disproportionately more expensive after the learning plane exists, which
is precisely when it becomes necessary. Doing it first is the cheap
ordering.

**Rely on the type boundary alone.** Retracted in ADR-002 D2; Python has
no private constructors.

**Skip the capability token now that the process boundary carries the
trust.** Retained anyway, at reduced status: it correlates an authorised
decision to the effect that claims it, which is an audit property the
process boundary does not provide.
