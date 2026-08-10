# Core Episode Schema v0.1

```text
STATUS      draft specification, no implementation
ORDERING    defined BEFORE any executor exists
RATIONALE   we decide what evidence autonomous execution must produce
            before autonomous execution is capable of producing anything
SUPERSEDES  nothing; first schema document
```

An **episode** is one governed unit of delivery work: an intent, the
authority under which it ran, everything it did, what it cost, and what
is known about whether it worked. It is the only record from which Core
may learn.

---

## 1. Why this document exists before the executor

Retrofitting a record onto a running executor is how required fields
become optional. But there is a second reason that emerged from the
kernel/worker decision (ADR-003) and is worth stating because it
constrains every field below:

**With the gate in a separate process, the episode schema is not just a
record — it is the wire format.** Everything crossing the kernel/worker
boundary must be serialisable, addressable and independently
interpretable by a process that does not share memory with the thing
that produced it. A schema designed as an in-process object graph would
have to be rewritten. A schema designed as a wire format works for both.

Every field below is therefore JSON-representable, and every reference
between records is an id, never a pointer.

## 2. Record kinds and axes

> **REOPENED on independent review — this section is now known wrong.**
> The caveat in §10 Q1 was raised and then not acted on. Pushed, the
> conclusion is forced: `authority` is not orthogonal across all five
> kinds. `OBSERVATION + APPROVED` has no meaning; `EVIDENCE + REVOKED`
> is worse. One universal record with one universal authority axis is
> the wrong shape, and keeping it would repeat the exact error already
> rejected in the four-dimension proposal — a sum type modelled as a
> product type.
>
> The replacement is a **tagged union** plus separate entities:
>
> ```text
> Observation   provenance, observed_at, validity
> Inference     derived_from, validity
> Requirement   authority_ref, validity
> Decision      authority_ref, validity
>
> EvidenceArtifact  producer, digest, subject_refs, relation, validity
>                   relations: SUPPORTS | FALSIFIES | VERIFIES | DERIVED_FROM
>                   -- has provenance and validity; has NO authority axis
>
> AuthorizationEvent / AuthorityGrant / AuthorityRevocation
>                   -- authority is an EVENT, not a field on a record
> ```
>
> Note what this does to §2.1: if authority is an event stream rather
> than a mutable field, "authority-at-time vs authority-now" stops being
> a rule to remember and becomes a consequence of the representation.
> The record holds `authority_ref`; the events say what happened to it.
> That is the better fix and it supersedes §2.1's prose rule.
>
> v0.2 must adopt the tagged union. The flat form below is retained
> only so the change is legible.

Adopting the refinement in your §7, with one caveat recorded in §10.

```text
kind        OBSERVATION | INFERENCE | DECISION | REQUIREMENT | EVIDENCE
authority   INFORMATIONAL | APPROVED | REJECTED | REVOKED
validity    ACTIVE | STALE | SUPERSEDED | INVALID
```

`kind` is a discriminator — exactly one, never a lifecycle.
`authority` and `validity` are orthogonal axes over it.

```yaml
kind: DECISION            # kind: INFERENCE            # kind: OBSERVATION
statement: "Use Postgres" # statement: "repo modular"  # statement: "CI #174 failed"
authority: APPROVED       # authority: INFORMATIONAL   # authority: INFORMATIONAL
validity: ACTIVE          # validity: ACTIVE           # validity: ACTIVE
```

### 2.1 Authority-at-time vs authority-now

`authority` is mutable; an episode record is not. An action authorised
at execution and `REVOKED` an hour later did still happen under
authority, and the record must not be rewritten to say otherwise.

```text
episode record       append-only; stores authority AT THE MOMENT of action
project state item   mutable; carries authority NOW
```

These are two different objects and the schema keeps them apart.
Revocation appends a new record; it never edits an old one. Without this
rule, revocation silently corrupts the learning corpus — every past
episode involving a since-revoked capability would read as though it had
been unauthorised all along.

## 3. Episode structure

```yaml
episode:
  # --- Identity -------------------------------------------------------
  episode_id:          str        # content-addressed, stable
  parent_episode_id:   str | null # RESERVED: always null in Core v1 (ADR-002 D3)
  authority_scope:     object     # RESERVED: delegated-authority envelope
  started_at:          iso8601
  ended_at:            iso8601 | null

  # --- Provenance keys (see §4) ---------------------------------------
  provenance:
    core_version:            str
    harness_version:         str
    model_provider:          str
    model_identifier:        str
    model_version:           str | null   # null when the provider hides it
    capability_set_digest:   str
    routing_policy_version:  str
    skill_versions:          {name: version}
    environment_digest:      str          # OS, toolchain, repo commit

  # --- Intent ---------------------------------------------------------
  intent:
    request:            str
    classification:     INFO | MICRO | MATERIAL | PROGRAM
    success_criteria:   [str]              # stated BEFORE execution
    granted_authority:  object

  # --- Context --------------------------------------------------------
  context:
    manifest:        [{source_id, kind, digest, bytes}]
    provenance:      [{source_id, origin, retrieved_at}]
    token_usage:     {input, output, cached}

  # --- The governed sequence ------------------------------------------
  decisions:          [Decision]
  action_proposals:   [ActionProposal]
  authorizations:     [AuthorizationDecision]  # proposal_id -> ALLOW/DENY/APPROVAL
  effects:            [Effect]                 # each carries authorization_id
  evidence:           [Evidence]               # content-addressed
  verification:       [VerificationResult]
  human_interventions:[Intervention]

  # --- Accounting -----------------------------------------------------
  cost:
    tokens:      int
    wall_ms:     int
    tool_calls:  int
    currency:    {amount, unit} | null

  # --- Outcome --------------------------------------------------------
  closure:
    status:      PASS | FAIL | PARTIAL | ABANDONED
    at:          iso8601
    rationale:   str

  # --- Computed, never asserted (see §5) -------------------------------
  learning_eligibility:
    verdict:     ELIGIBLE_POSITIVE | ELIGIBLE_NEGATIVE | INELIGIBLE | UNRESOLVED
    conditions:  {condition_name: PASS | FAIL | UNDECIDABLE}
    computed_by: str          # checker version; never a model
```

### 3.1 The structural invariants

Three relations must hold, and are checkable without judgment:

```text
every Effect            -> has an authorization_id resolving to an ALLOW
every AuthorizationDec. -> has a proposal_id resolving to an ActionProposal
no Effect               -> follows a DENY for the same proposal_id
```

An episode violating any of these is not merely ineligible for learning
— it is evidence of a gate bypass and must raise, not record quietly.

## 4. Provenance keys exist because learning is per-model

HarnessBank reports that harness gains are model-specific rather than
universally optimal; Self-Harness mines weaknesses per model family.
So a learned rule is never a global fact, and the fields that scope it
have to be present from the first episode ever recorded — they cannot be
backfilled, and their absence makes later experiments uninterpretable
rather than merely weaker.

```yaml
learning_scope:
  model_family:      required
  model_version:     required when the provider exposes it
  harness_version:   required
  capability_set:    required
  task_class:        required
```

Promotion scopes, narrowest first:

```text
PROJECT
REPOSITORY
TASK_CLASS
MODEL_VERSION
MODEL_FAMILY
GLOBAL          <- requires evidence at every narrower scope it generalises
```

`GLOBAL` is deliberately the hardest to reach. A rule promoted globally
on evidence from one model family is the exact error HarnessBank warns
about.

## 5. Learning eligibility — mechanically decidable, or UNRESOLVED

> **AMENDED — split into three questions, not one.** The section below
> conflates *can this episode enter the corpus* with *can this episode
> support this conclusion*. Only the first is mechanical.
>
> ```text
> A  CORPUS ADMISSION    fully deterministic; the table below
>                        -> ADMISSIBLE_POSITIVE | ADMISSIBLE_NEGATIVE
>                           INADMISSIBLE | UNRESOLVED
> B  PROPOSAL GENERATION model-driven and allowed to be wrong; a bad
>                        diagnosis only yields a candidate the gate rejects
> C  PROMOTION CREDIT    fresh prospective evaluation, candidate vs frozen
>                        baseline, same versioned environment
> ```
>
> This dissolves the environment-comparability problem rather than
> displacing it, which is what the §5 rewrite actually did. **Historical
> episodes are proposal substrate; credit comes from fresh held-out
> evaluation.** Stop asking historical traces to carry causal weight they
> cannot bear. The verdict names below should become `ADMISSIBLE_*` in
> v0.2, since "eligible" was overloaded to mean both admission and
> credit.

This is the section I would most want an independent reviewer to attack,
because it is where a judgment call would quietly reintroduce
self-certification.

**Rule: every condition is a deterministic function of recorded fields.
No condition may be evaluated by a model.** A condition that needs
judgment resolves to `UNDECIDABLE`, and any `UNDECIDABLE` forces the
episode to `UNRESOLVED` — never to eligible.

| Condition | Mechanical rule |
|---|---|
| `outcome_resolved` | `closure.status ∈ {PASS, FAIL}` and `closure.at` set |
| `verification_sufficient` | `max(verification.strength) ≥ required_for(intent.classification)` |
| `trace_complete` | all three §3.1 relations hold; no gaps in event sequence |
| `identity_recorded` | every non-nullable `provenance` field present |
| `intervention_explained` | every `Intervention` has `actor`, `at`, `reason` |
| `environment_recorded` | `environment_digest` present |
| `no_evidence_gap` | every `VerificationResult` has a resolvable `evidence_ref` |
| `no_policy_violation` | no `Effect` following a `DENY` for its proposal |

Two of your eight conditions were rewritten rather than adopted:

```text
"environment comparable"  -> environment_recorded
   comparability is a property of a PAIR of episodes and cannot be
   decided about one episode in isolation; it is evaluated at learning
   time, against the candidate's own environment

"no unexplained intervention" -> intervention_explained
   "unexplained" invites a model to judge whether a reason is adequate;
   the checkable property is that a reason was recorded at all
```

### 5.1 Verification strength

```text
NONE         no check ran
ASSERTED     a model or human stated the outcome      <- NEVER sufficient
CHECKED      a deterministic check ran and passed
INDEPENDENT  the check was not authored in this episode
ADVERSARIAL  mutation or negative cases were exercised
```

`ASSERTED` never satisfies `verification_sufficient` at any tier. This
is v1's truthful-closure rule — model assertion is not a PASS — carried
into the learning plane unchanged.

### 5.2 Verdicts

```text
ELIGIBLE_POSITIVE   all conditions PASS, closure = PASS
ELIGIBLE_NEGATIVE   all conditions PASS, closure = FAIL
INELIGIBLE          any condition FAIL
UNRESOLVED          any condition UNDECIDABLE, or closure ∈ {PARTIAL, ABANDONED}
```

`learning_eligible ≠ success`, in both directions. A verified failure is
`ELIGIBLE_NEGATIVE` and is among the most valuable records Core can
hold. An unverified success is `INELIGIBLE` and is actively dangerous —
it teaches a pattern nothing confirmed.

This is also why the dataset argument is quality-first: ten
`ELIGIBLE_NEGATIVE` episodes with complete traces outrank several
thousand `INELIGIBLE` successes, which contribute nothing and cost
storage.

## 6. Mutable zones

> **AMENDED — "verification procedures" was too broad for Zone A** (this
> was Q7 in the handoff and the answer is that it does not survive).
> Verification splits across all three zones, governed by a monotonic
> rule:
>
> ```text
> A  verification suggestions, test selection hints, optional extra
>    checks, diagnostic probes
> B  verifier implementations, verification adapters and orchestration,
>    new testing strategies
> C  what evidence counts, minimum required verification, closure
>    semantics, eligibility semantics, promotion sufficiency
> ```
>
> **Ordinary learning may add optional verification. It may never remove
> or weaken constitutionally required verification.** Without that rule
> Zone A rewrites Zone C by the back door: a learned change to a
> verification procedure changes what counts as verified, which changes
> admission, which changes what may teach Core.

```text
ZONE A  self-improvable
        skills, prompts, routing hints, context heuristics,
        decomposition patterns, verification SUGGESTIONS, tool strategies

ZONE B  restricted evolution -- stronger evaluation required
        routing algorithms, learning algorithm, budget allocation,
        execution-strategy policy

ZONE C  constitutional -- no ordinary self-promotion
        authority semantics, Effect Gate, kernel/worker separation,
        event integrity, credential access, promotion ownership,
        PROMOTION THRESHOLDS, maximum privilege rules
```

**`promotion thresholds` moved from Zone B to Zone C.** A threshold is
the promotion criterion. Letting the improver lower the bar it must
clear is materially identical to letting it own promotion, which Zone C
already forbids — and the change would be self-ratifying, since the
evaluation certifying the new threshold runs under the old one exactly
once and every evaluation afterwards runs under the new one. This is the
same shape as the Continuum comparator back door, where a `floor=`
parameter let a caller choose the bar it was measured against.

## 7. Improvement lifecycle

```text
PROPOSED -> EVALUATING -> ACCEPTED  -> PROMOTED -> RETIRED
                       -> REJECTED               -> ROLLED_BACK
```

Ordinary state, deliberately not a strategy ensemble. The pipeline:

```text
EPISODES (eligible only)
   -> DIAGNOSER          (may be a model; proposes only)
   -> IMPROVEMENT PROPOSAL
   -> SANDBOX
   -> REPLAY
   -> REGRESSION
   -> MUTATIONS
   -> HELD-OUT EVAL
   -> SECURITY CHECK
   -> PROMOTION DECISION (never the proposer)
```

A proposal touching Zone C is rejected by construction at the pipeline
entrance, before evaluation — not scored and found wanting. Evaluating
a constitutional change on its merits already concedes that merit is the
criterion.

## 8. What v0.1 deliberately does not define

```text
the executor                    ADR-001 ordering
the model adapter               ADR-001 ordering
recursion beyond reserved keys  ADR-002 D3
the learning algorithm          out of scope until K and E ship
the diagnoser                   out of scope
storage format and indices      premature
```

## 9. Immediate follow-on

A schema whose invariants exist only in prose is the failure this
project keeps catching. v0.2 should ship:

1. a JSON Schema for the episode record;
2. a validator for the three §3.1 relations;
3. the eligibility checker, written to accept a recorded episode and
   emit a verdict — **authored independently of whatever writes
   episodes**, per the standing rule that self-authored verification
   fails systematically.

None of these is an execution path, so none is gated by ADR-001.

## 10. Open questions, surfaced rather than decided

**Q1 — is `EVIDENCE` a kind or a relation?** As a peer `kind`, evidence
inherits the `authority` axis, which is meaningless for it: evidence is
never `APPROVED`, only valid or not. `validity` does apply — evidence
referencing a superseded commit goes `STALE`. So either `authority` is
N/A for one of five kinds, which weakens "orthogonal", or evidence is a
relation (`evidence_for: record_id`) rather than a kind. This is the
same shape as the unresolved `OBSERVATION`/`AUTHORIZED` tension in gap
analysis §E, and the two should be resolved together.

**Q2 — what makes two environments comparable?** §5 defers this to
learning time, which is correct but not free: something must eventually
decide whether an episode from a different toolchain may inform a
candidate. If that decision is a model's, self-certification returns
through the side door.

**Q3 — who authors the eligibility checker?** It must not be whatever
writes episodes. In Continuum the author of a checker passed the exact
defect it was built to catch. That was one person; here it is one model,
which is not obviously better.
