# ADR-003 — Trusted kernel, untrusted worker

```text
STATUS    Accepted, with two unresolved design problems (§4, §5)
DATE      2026-08-09
AMENDS    ADR-002 D2 (graduated enforcement)
SUPPORTS  ADR-001 (governance precedes execution)
```

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
