# Escapement Core — design handoff for independent review

```text
PURPOSE   independent review of a set of design decisions taken but
          NOT yet implemented
STATE     zero implementation; four documents and three ADRs
ASK       what conclusions are entailed by these decisions, including
          ones we did not draw and would not like
```

You are reviewing an architecture that has not been built. Nothing here
is load-bearing yet, so a finding now costs a document edit rather than
a migration. Please treat the decisions as claims to be attacked, not as
context to be accepted.

**Framing note, deliberately stated.** Do not try to confirm that these
decisions are sound. Ask what follows from the artifacts, including
conclusions the authors did not reach. A previous phase of this project
(Escapement-Continuum) ended in a negative result — its distinguishing
architectural claim did not survive adversarial specification — and the
main reason it was caught was reviewers who refused the framing offered
to them.

---

## 1. What Escapement is

Three repositories, one lineage.

```text
Escapement v1     public, stable, in use. A governance harness for AI
                  coding work: lifecycle Specify -> Route -> Execute ->
                  Verify -> Persist; task tiers INFO/MICRO/MATERIAL/
                  PROGRAM; capability routing; content-addressed check
                  evidence; closure that refuses to report PASS on a
                  model's assertion or a critical failure.
                  122/122 routing evals, 189 unit tests, doctor clean.

Escapement-       public, research line. Phase I concluded with a
Continuum         NEGATIVE result and is frozen. Its lessons, not its
                  code, carry forward.

Escapement-Core   private, new. Cloned from v1 with full history, no
                  GitHub fork relationship. The subject of this review.
```

**The single most important fact about the starting position:** v1 has
no model-to-effect path at all. `grep -rniE "subprocess|os\.system|
exec\("` across the runtime and router returns nothing. Core is
therefore being built in the unusual position of adding governance
*before* adding execution, rather than retrofitting it after.

## 2. What Core is trying to be

After a landscape review of 2025–August 2026, the conclusion was that no
individual capability is differentiating. Verified prior art:

```text
Parallax (2604.12986)          cognitive-executive separation; an
                               independent validator between reasoning
                               and execution. 98.9% of 280 attacks.
Unfireable Safety Kernel       external authorization kernel: process
(2606.26057)                   separation, fail-closed, signed evidence.
                               704/704 blocked over 1,000 self-mods.
Microsoft AGT (GitHub)         generic agent governance: policy, SPIFFE
                               identity, sandboxing, MCP gateway,
                               approvals, delegation, Decision BOM.
                               5 languages, 992 conformance tests.
Shepherd (2605.10913)          Git-like reversible execution traces,
                               forkable, 5x faster than docker commit.
lambda-RLM (2603.20105)        typed lambda-calculus runtime replacing
                               free-form recursive code generation.
Self-Harness (2606.09498)      weakness mining from traces -> minimal
                               harness change -> regression validation.
HarnessBank (2607.13683)       gene-bank harness evolution with gated
                               verification. 5.1-15.4%, 7 benchmarks.
DGM (2505.22954)               open-ended self-rewriting coding agents.
```

All eight were fetched and verified rather than cited from memory.

The intended position is therefore a **combination**, stated as:

> A governed, recursive-capable, self-improving software-delivery
> runtime in which model reasoning produces proposals, executable
> authority is structurally separate, every consequential action
> produces verifiable evidence, and only independently verified
> experience may modify future behaviour.

Compressed: *a self-improving software agent that cannot self-certify
or self-authorize.*

## 3. The decisions taken

**D1 — six responsibilities, three new.**

```text
INHERITED  L delivery lifecycle
           V verification and evidence-backed closure
           T test discipline (causal sensitivity, mutation, independent
             verification)
NEW        K constitutional kernel (authority, effect gate, promotion
             boundary)
           E episode and evidence model
           P learning and promotion
```

Trust flows K → P → E and never back. Everything else — orchestration
(LangGraph), policy breadth (AGT), replay semantics (Shepherd),
transport (MCP), typed recursion (λ-RLM) — is imported, not built.

**D2 (superseded) → D2' — the trust boundary is a process boundary.**

An earlier draft claimed a Python type made bypass structurally
impossible: `AuthorizedAction` with no public constructor, so
`executor.execute(proposal)` "must not type-check." This was retracted.
It fails mypy but not Python — `__new__`, `copy`, `dataclasses.replace`
and pickle all reach around a private constructor. A lint had been
presented as a structural guarantee.

The replacement is kernel/worker process separation, designed **before**
the first executor rather than deferred behind it, because
model-generated and learned code is a deliberate Core capability:

```text
TRUSTED KERNEL                  UNTRUSTED WORKER
policy, authority, effect gate  model, generated code, tools,
promotion gate, event integrity shell, MCP, learned strategies
credentials
```

Static types are demoted to ergonomics; HMAC capability tokens are
demoted to audit/dispatch correlation.

**D3 — recursion deferred, architecturally reserved, empirically
admitted later.** Not cut. It re-enters as a routed `ExecutionStrategy`
(`DIRECT | RLM | RECURSIVE_HARNESS`) that must beat the direct baseline
on verified outcomes per task class. `parent_episode_id` and
`authority_scope` are reserved from the first episode recorded.

**D4 — learning is per-model.** HarnessBank reports gains are
model-specific rather than universally optimal. Model identity keys the
episode; the promotion gate is per-model; promotion scopes run
`PROJECT → REPOSITORY → TASK_CLASS → MODEL_VERSION → MODEL_FAMILY →
GLOBAL`, with GLOBAL deliberately hardest to reach.

**D5 — terminology and claim discipline.** "Proof-carrying" retired for
"verified execution episodes" (proof-carrying code is a real formal
artifact and the comparison would be lost). The "dataset moat" framing
dropped. Claims split three ways: a product claim, an engineering
hypothesis (traces can drive harness adaptation without the adapter
controlling its own promotion criteria), and a research hypothesis
(governed self-improvement improves outcomes while preserving invariant
guarantees). Only the third needs serious empirical proof.

**D6 — state model.** One discriminator, two orthogonal axes. An earlier
four-dimension proposal was rejected because `epistemic` and `decision`
are alternatives, not coordinates — modelling them as two nullable
fields makes illegal states representable.

```text
kind       OBSERVATION | INFERENCE | DECISION | REQUIREMENT | EVIDENCE
authority  INFORMATIONAL | APPROVED | REJECTED | REVOKED
validity   ACTIVE | STALE | SUPERSEDED | INVALID
```

**D7 — learning eligibility is mechanically decidable or it is
UNRESOLVED.** Eight conditions, each a deterministic function of
recorded fields. No condition may be evaluated by a model. Verdicts:
`ELIGIBLE_POSITIVE`, `ELIGIBLE_NEGATIVE`, `INELIGIBLE`, `UNRESOLVED`.
Verification strength ladder `NONE < ASSERTED < CHECKED < INDEPENDENT <
ADVERSARIAL`, where `ASSERTED` is never sufficient at any tier.
`learning_eligible ≠ success` in both directions: a verified failure is
valuable, an unverified success is dangerous.

**D8 — three mutable zones.** A self-improvable (skills, prompts,
routing hints, context heuristics, verification procedures); B
restricted (routing algorithms, learning algorithm, budget allocation,
strategy policy); C constitutional (authority semantics, effect gate,
kernel/worker separation, event integrity, credentials, promotion
ownership, **promotion thresholds**, privilege rules). A proposal
touching Zone C is rejected at the pipeline entrance, not scored.

## 4. Where the authors already disagreed with themselves

Recorded because the disagreements may be more informative than the
conclusions.

```text
"typed boundary is Core's clearest differentiator"
   -> retracted; both novel and functional claims were false

"four orthogonal state dimensions"
   -> rejected; two of the four are alternatives, not coordinates

"process separation only once Core governs destructive effects"
   -> overruled; it is the boundary, so it comes first

"cut recursion"
   -> overruled; deferred and reserved, admitted later on evidence

"promotion thresholds are Zone B"
   -> moved to Zone C; a threshold IS the promotion criterion, and
      changing it would be self-ratifying

"the dataset is the moat"
   -> dropped; single-operator volume, single distribution, and per D4
      it does not transfer across models
```

## 5. Questions for you

Ordered by how much damage a wrong answer does.

1. **Is `K/E/P` actually a system, or three unrelated good ideas?** The
   claim is that the combination is the contribution. Attack that.
   Would removing any one leave something still coherent and useful? If
   so, is the removed one the actual product?

2. **Same-user process separation.** On Windows, a same-user process can
   `OpenProcess` a sibling with `PROCESS_VM_READ`. On Linux, yama
   `ptrace_scope ≥ 1` prevents a child tracing its parent. So the
   boundary's strength is deployment-dependent. Is "kernel and worker as
   different OS principals" the minimum bar, or is a container the only
   honest answer? Does a boundary this configuration-dependent deserve
   to be called constitutional?

3. **Credentials and tools are on opposite sides.** Kernel holds
   credentials; worker holds tools/MCP/shell. Tools need credentials.
   Options considered: (A) give the worker credentials — defeats the
   boundary; (B) kernel proxies every credentialed call — the kernel
   grows an egress stack for every protocol, enlarging the trusted
   computing base; (C) kernel mints short-lived narrowly-scoped
   per-action credentials — works for tokens Core issues, often
   impossible for a third-party API key. Is there a fourth option? Is C
   with a B fallback actually coherent, or does the fallback swallow the
   design?

4. **Can eligibility really be mechanical?** Two of the original eight
   conditions were rewritten because they required judgment
   ("environment comparable" → "environment recorded"; "no unexplained
   intervention" → "intervention explained"). Is that a genuine fix or
   is it displacement — moving the judgment to learning time, where it
   is less visible? Something must eventually decide whether an episode
   from a different toolchain may inform a candidate. If a model
   decides, self-certification returns through a side door.

5. **Is `EVIDENCE` a kind or a relation?** As a peer `kind` it inherits
   the `authority` axis, which is meaningless for it — evidence is never
   `APPROVED`. `validity` does apply. Either "orthogonal" is false for
   one of five kinds, or evidence should be `evidence_for: record_id`.
   The same shape recurs for `OBSERVATION`: can an observation ever be
   `APPROVED`? It is currently always `INFORMATIONAL`, so the field
   carries no information, and it was retained only so that no code path
   can forget to check and no missing field can read as authorised.

6. **Who authors the eligibility checker?** It must not be whatever
   writes episodes. In Continuum, an author's own contract checker
   passed the exact defect it was written to catch — it scored a
   hardcoded comparator as PASS. That was one person. Here it would be
   one model, which is not obviously better. Is there a construction
   that makes this structural rather than procedural?

7. **Does Zone A survive contact with Zone C?** Zone A includes
   "verification procedures." Zone C includes "promotion ownership." A
   learned change to a verification procedure changes what counts as
   verified, which changes eligibility, which changes what may teach
   Core. Is the A/C line drawn in the right place, or does
   "verification procedures" belong in B or C?

8. **Is the engineering hypothesis falsifiable as stated?** "Verified
   execution traces can drive useful harness adaptation without the
   adaptation process controlling its own promotion criteria." What
   observation would falsify it? If no failure condition can be stated,
   it is a description, not a hypothesis — and that is precisely how
   Continuum's Phase I claim survived as long as it did.

9. **What is missing?** The reduction removed a great deal. Name
   anything cut that will have to come back, and anything retained that
   should not have been.

10. **The strongest case against the whole thing.** If Core should not
    be built — if AGT plus Claude Code plus a discipline document
    genuinely covers it — make that argument as well as it can be made.

## 6. What would be most useful back

Not a verdict on whether this is good. Specifically:

```text
- claims that do not follow from their stated evidence
- invariants that are prose rather than structure
- any place where a model decides something the design says only
  deterministic code may decide
- the cheapest experiment that could falsify the engineering hypothesis
- what an adversary with worker-process code execution actually gets
```

If the honest conclusion is that K/E/P is not a system, or that the
process boundary is not defensible on a single machine, that is a more
valuable result than a list of improvements — and it arrives before
anything has been built, which is the cheapest moment it can arrive.
