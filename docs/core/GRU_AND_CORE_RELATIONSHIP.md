# GRU and Escapement Core — what the relationship actually is

```text
STATUS    analysis. No decision is binding until an ADR adopts it.
DATE      2026-09-15
INPUTS    GRU Product and Architecture Freeze v1.1 (1180 lines)
          docs/decisions/ADR-001 .. ADR-005
          docs/core/*.md  (six design documents, zero implementation)
QUESTION  does GRU supersede Core, absorb it, take shape from it,
          or run alongside it?
ANSWER    all three of the first, in different layers. Not the fourth.
```

Every claim below is marked. **FACT** is stated in one of the source
documents and cited. **INFERENCE** is a reading of those documents that
the documents do not state. **RECOMMENDATION** is a proposal for action.
Where the evidence does not decide something, it says so rather than
picking the comfortable side.

The governing constraint on this document is GRU §35: *"Do not keep
Escapement code because it is ours."* Continuum Phase I consumed a year
on a research line kept alive by that exact bias. The test applied
throughout is not "is this good work" but "would a blank-page GRU write
this, and if not, does it still need it".

---

## 0. The fact that reframes the whole question

**FACT.** Escapement Core has no implementation. ADR-005 D1 freezes it
at a commit with "no executor, kernel, broker, IPC or MCP surface", and
the README states "no Core implementation exists yet". The six documents
in `docs/core/` and five ADRs are the entire output.

**INFERENCE.** The language migration therefore costs Core nothing.
Python → TypeScript destroys an inheritance that Core never had. What it
destroys is *v1's*: ADR-002 D1 claims three of Core's six
responsibilities — `L` delivery lifecycle, `V` verification and closure,
`T` test discipline — are "already earned by v1, migrated not
reinvented". Migration is exactly what GRU cannot do. A TypeScript GRU
inherits those three as *prose*, at the same maturity as `K` and `E`.

So GRU §35's rule does not bite Core at all. Core has no code to keep.
It bites ADR-002 D1, which counts a Python codebase as paid-for work.

```text
WHAT THE MIGRATION KILLS   v1's 296 files, 189 tests, 122 evals as an
                           inheritance -- L, V, T revert to specification

WHAT THE MIGRATION SPARES  every Core artifact, because all of them are
                           already language-neutral prose and JSON
```

This is the single most consequential asymmetry in the comparison and
neither document states it.

---

## 1. Concept correspondence

### 1.1 Core → GRU

| Core concept | GRU concept | Match | Note |
|---|---|---|---|
| `K` constitutional kernel (ADR-002 D1) | AVL, "GRU's constitutional layer" (§14) | **PARTIAL** | Same slot, same authority claim ("Director owns decisions, AVL owns truth" ≈ Core's authority/evidence split). AVL's specification is ~30 lines; `K`'s is a threat model, ten properties and an adversary. GRU names the box; Core filled it |
| `K_semantics` | "AVL project-consequence semantics" → BUILD NATIVE (§17.1); "Project constitution → BUILD GRU SEMANTICS" (§20) | **EXACT** | GRU independently reaches the semantics/enforcement split and freezes the semantics side (§34) while leaving "Cedar / AGT / other policy plumbing" provisional |
| `K_enforcement` | OpenShell "STRONG OPTIONAL BACKEND", Cedar "CANDIDATE DEPENDENCY" (§20) | **EXACT in shape, CONTRADICTORY in posture** | Both push enforcement to replaceable backends. ADR-003 §6 requires a fail-closed rule when no backend satisfies the property; GRU has no such rule and calls the sandbox *optional*. See §3.2 |
| `E` episode record | "Verified Episodes" + "Event History" in Minion Mind (§13) | **PARTIAL** | GRU lists both as contents of a store. No schema, no fields, no invariants. `CORE_EPISODE_SCHEMA_v0.1` is the only specification either project has |
| Learning-eligibility predicate (schema §5) | "learning and promotion eligibility" — owned by AVL (§14) | **PARTIAL** | GRU names the responsibility and assigns it correctly. It defines nothing. Core's eight mechanical conditions and the `UNRESOLVED`-on-`UNDECIDABLE` rule have no GRU counterpart |
| `P` learning and promotion | §29 organizational learning | **PARTIAL, converging** | The loops are the same shape: proposal → prospective evaluation → independent verification → PROMOTE/REJECT → monitor → RETAIN/ROLL BACK. GRU §29 ends with "The proposer cannot weaken the verification criteria that decide whether its own proposal is promoted" — that is Core's P6 and P7 in one sentence, reached independently. Missing in GRU: per-model scoping (ADR-002 D4), promotion-scope ladder (schema §4), Zone C |
| `L` delivery lifecycle | Project Contract, Organization Compiler (§11), GRU's PM duties (§8) | **PARTIAL** | Same function, different decomposition. GRU adds staffing and organisation compilation, which Core has nothing resembling. Core's phase machine is v1's and does not carry |
| `V` verification / evidence-backed closure | EvidenceRequirement, VerificationContract (§14); independent verification (§12.1, §4.9) | **PARTIAL** | Principle identical and stated forcefully on both sides. GRU has no evidence *artifact* model — no digests, no content addressing, no producer/relation fields. Core's `EvidenceArtifact` is strictly more specific |
| Verification strength ladder `NONE < ASSERTED < CHECKED < INDEPENDENT < ADVERSARIAL` (schema §5.1) | binary verification status + "independent verification where materiality requires it" (§12.1) | **PARTIAL** | GRU has two of five rungs and no rule that `ASSERTED` is never sufficient. Ten lines of Core prose that GRU is missing outright |
| `T` test discipline — causal sensitivity, mutation registry, independent review | — | **NO-EQUIVALENT** | GRU §32.2 tests *seams*; nothing in GRU tests whether a test can fail. The mutation registry (gap analysis §H) exists because Continuum shipped a checker that passed the defect it was written to catch. GRU has no defence against that class |
| Effect classes: `FILESYSTEM_READ/WRITE`, `PROCESS_EXECUTION`, `NETWORK`, `CREDENTIAL_ACCESS`, `REPOSITORY_MUTATION`, `EXTERNAL_SIDE_EFFECT`, `DESTRUCTIVE` (audit) | ProjectConsequence (§14.1) | **PARTIAL — different altitude, and GRU's is better** | GRU normalises to *project* consequence: "different technical routes that produce the same material project consequence should carry the same project-delivery obligation". Core's classes are route-level. They compose — Core's taxonomy is the input to GRU's normalisation, and GRU §14.1 does not say how that normalisation is computed |
| T0 operator | Project Director (§6, §7) | **EXACT** | |
| T1 constitutional control | AVL (§14) | **EXACT** | |
| T2 privileged capability providers | host sandbox, OpenShell, Cedar, AGT, provider APIs (§20) | **EXACT** | Core's "trusted to enforce what they claim, not trusted to understand Escapement semantics" transfers verbatim |
| T3 untrusted intelligence | Minions — **and GRU, DRU, Nefario** | **PARTIAL, and this is a finding** | See §1.3 |
| T4 untrusted input | §14.2 "Information is not authority. Untrusted information can influence reasoning. It cannot grant itself permission." | **EXACT** | Both projects state the rule and neither models prompt injection. Core threat model §6 admits it; GRU does not |
| Kernel/worker process separation (ADR-003) | — | **NO-EQUIVALENT** | §5 draws AVL as a layer in a stack. Nothing says it is a boundary, a process, or a different principal. See §3.2 |
| Credential broker (ADR-003 §6) | §14.3 "capability handles and model routes, not raw secrets, wherever the underlying substrate allows" | **PARTIAL** | Same design. GRU's trailing clause is a discretionary escape that Core's fail-closed rule forbids |
| Zones A / B / C with promotion thresholds in C (schema §6) | §34 freeze boundary table; §17.1 BUILD NATIVE list | **NO-EQUIVALENT — and not yet needed** | GRU's freeze table governs *humans revising the architecture*. Zones govern *the system revising itself*. GRU §31.2 defers autonomous self-modification entirely, so the gap is correctly empty today |
| Reduction ladder `normalize → hook → broker → own` | four-way sourcing doctrine `DEFER → INTEGRATE → INSPIRE+REIMPLEMENT → BUILD NATIVE` (§17) | **PARTIAL — close cousins** | Same first-rung-that-holds structure. GRU's is a component-sourcing decision made up front; Core's is a post-failure remedy ladder with the rule that the "must own?" column stays `TBD` until the ladder is walked. That sequencing rule is the transferable delta |
| EXPERIMENT_CORE_000 preregistration | §32.1 seven validation experiments | **PARTIAL** | GRU §32.1 item 4 — "Can the same material consequence reached through different technical routes receive equivalent authority/evidence treatment?" — *is* Core's S1 and S7 restated. GRU states the question and supplies no control, no adversary, no predictions, no scoring rule and no criterion for a well-run experiment |
| State model: one discriminator + two orthogonal axes (gap analysis §E) | claim states `SUPPORTED / CONTESTED / REJECTED / SUPERSEDED / UNVERIFIED` (§13.1) | **PARTIAL — and Core has the correction GRU needs** | GRU's five states mix evidential support (four of them) with lifecycle validity (`SUPERSEDED`). That is the sum-type-as-product-type collapse gap analysis §E spent a section rejecting. A claim can be `SUPPORTED` *and* `SUPERSEDED`; GRU's enum cannot say so |
| Per-model learning (ADR-002 D4) | §29 "Which model routes worked best for each role or task?" | **PARTIAL** | GRU asks the question. Core supplies the rule: model identity is part of the episode key, and an improvement validated on one model does not promote for another without its own evidence |
| Verified determinization | §30, with `ACTIVE / STALE / SUPERSEDED` lifecycle | **NO CORE EQUIVALENT** | Reverse absence. See §1.2 |
| "Agent API / MCP / CLI over one kernel" (ADR-005 D4) | Workbench / CLI / web shell over AVL (§25.3) | **PARTIAL** | Same shape, reached independently |

### 1.2 GRU concepts with no Core equivalent

Absences run both ways, and listing only one direction would flatter Core.

```text
DRU                        Core has "the author does not verify their own
                           work". It has no persistent adversarial role,
                           no bounded debate, no residual-disagreement
                           preservation. GRU §9-§10 is entirely new.

DirectorOverride           GRU §7: a valid override records
                           Verification: FAIL / Director: PROCEED /
                           Residual Risk: ACCEPTED. Core has no override
                           concept at all -- only a project-level
                           implementation_authorized boolean.

Organization Compiler      Core has no staffing model, no role
                           derivation, no team topology.

Dr. Nefario                Core has no capability-acquisition role. v1
                           has a capability registry; nothing admits,
                           provisions, health-checks or retires.

Capability readiness       GRU §16's six health layers (install, startup,
                           protocol, semantic, resource, security) have
                           no Core counterpart.

ExecutionRoute + health    GRU §23 treats provider/model/credential/
                           endpoint as one object with multidimensional
                           health. Core's episode schema records model
                           identity for learning scope only.

Verified determinization   GRU §30. Core deliberately excluded it.

Workbench                  Core has no product surface at all.
```

**INFERENCE.** Core is narrower than GRU by design and deeper in exactly
one place. GRU is a delivery organisation with a constitutional layer
sketched in; Core is a constitutional layer with no organisation. They
are not competing designs of the same thing.

### 1.3 The trust-class finding

**FACT.** GRU §5 draws the organisation as a stack: Director → GRU/DRU →
Nefario → Minions → Minion Mind → AVL → chassis. **FACT.** GRU §14 says
"GRU has managerial authority. AVL has constitutional authority."

**INFERENCE.** The rule is right and the diagram contradicts it. Under
Core's threat model, GRU, DRU and Nefario are all **T3** — they are
model-driven roles, and "assume arbitrary code execution inside T3" does
not exempt a role because it is senior. Only the Director (T0) and AVL
(T1) sit outside T3. Drawing AVL *below* the managers invites the
reading that the managers are trusted and AVL is plumbing they call.

Two specific consequences GRU does not address:

```text
NEFARIO IS T3 PERFORMING T2 ADMISSION

GRU §28: "Capability discovery does not imply installation. Installation
does not imply authorization." Correct, and it names no authoriser.
Nefario -- untrusted intelligence -- runs the ADMIT step (§15) that
decides which privileged capability providers enter the system. That is
a privilege-escalation seam. Core's threat model would require the
admission decision to sit in T1 or T0, not T3.

DRU IS NOT AN INDEPENDENT VERIFIER IN THE T-CLASS SENSE

GRU §9 makes DRU independent of GRU. Both are T3. Adversarial challenge
between two untrusted components raises quality; it does not satisfy
P6/P7, which require the certifier to be outside the class that
proposes. GRU §29's promotion rule is correct precisely because it does
NOT route through DRU.
```

**RECOMMENDATION.** One diagram edit and one sentence in GRU §14 fix
both. This is the cheapest high-value transfer in the whole corpus.

---

## 2. What Core produced that GRU can use without any code

Assessed against one test: does it survive a language change, and does
GRU currently lack it?

| Artifact | Source | Transfers | Why |
|---|---|---|---|
| Trust classes T0–T4, one adversary, properties P1–P10 | `CORE_THREAT_MODEL_v0.1.md` §1–§3 | **YES — highest value** | GRU has no adversary model. Mechanism-agnostic by construction: "Technologies are evaluated against these, never substituted for them." Needs one edit (§1.3) |
| Verification strength ladder + `ASSERTED` never sufficient | `CORE_EPISODE_SCHEMA_v0.1.md` §5.1 | **YES** | Ten lines. Composes cleanly with GRU §7's Director override: `FAIL` + `PROCEED` + recorded residual risk is truthful; `ASSERTED` → `PASS` is not |
| Episode schema and its learning-eligibility predicate | `CORE_EPISODE_SCHEMA_v0.1.md` §3, §5 | **YES, with rework** | The only episode specification either project has. Language-neutral by accident: ADR-003 forced it to be a *wire format* — "every field JSON-representable, every reference an id, never a pointer" — for a kernel/worker boundary GRU does not have. The constraint outlived its reason. Rework required: §2 is self-declared wrong (flat record must become a tagged union) |
| Effect surface audit method | `EFFECT_SURFACE_AUDIT.md` "Method, for repetition" | **YES — and more needed in GRU than in Core** | The audit's finding was that host permissions mediate the host's own tools, not 28 CLI subcommands an agent reaches through a shell. GRU's route surface is larger by an order of magnitude: chassis terminals, workspaces, MCP tools, plugins, and a Nefario that installs arbitrary capabilities. This method is how you enumerate the routes GRU §32.1 item 4 quantifies over |
| Reduction ladder, and the rule that a failure authorises nothing | `EXPERIMENT_CORE_000_PREREGISTRATION.md` "Scoring and the reduction ladder" | **PARTIAL** | GRU §17 already has the sourcing doctrine. The delta is the discipline: walk the ladder only *after* a failure, stop at the first rung, and keep the "must own?" column `TBD` until then |
| Preregistration discipline; route-bypass invariants S1–S8 | `EXPERIMENT_CORE_000_PREREGISTRATION.md` | **METHOD YES, PREDICTIONS NO** | The invariants map straight onto GRU §32.1 item 4 and §32.2. The predictions do not transfer — their control is v1 + Claude Code + AGT, which is not GRU's stack. The single best line in the corpus: "at least one prediction is wrong — if all nine match, suspect that the invariants were written to be predictable rather than diagnostic" |
| Credential broker + `IsolationBackend` as a *property* not a mechanism | ADR-003 §6 | **YES** | Maps onto GRU §14.3 and §20's OpenShell entry, and is more specific than either. Includes the fail-closed rule GRU lacks |
| State model: one discriminator, two orthogonal axes | `CORE_READINESS_GAP_ANALYSIS.md` §E | **YES** | Applies directly to GRU §13.1's claim enum, which commits the error §E rejects |
| Zones A / B / C, thresholds in Zone C | `CORE_EPISODE_SCHEMA_v0.1.md` §6 | **RESERVE, DO NOT PORT NOW** | GRU §31.2 defers autonomous self-modification. Porting a self-modification boundary before self-modification exists is scaffolding for later, which is the thing to skip. File it against the day §29 acquires a runtime |
| Per-model promotion scoping | ADR-002 D4; schema §4 | **YES** | "Core got better" is never a global statement. Cheap now, expensive to retrofit — the scope fields cannot be backfilled |
| Retraction-in-place documentation convention | every Core document | **YES, as a convention** | `CORRECTED` / `REOPENED` / `AMENDED` blocks that leave the falsified claim visible. GRU §1.1's freeze table does versions, not retractions |

### 2.1 One thing the migration makes *better*

**INFERENCE, with a stated ceiling.** ARCHITECTURE_REDUCTION §2 retracted
the typed effect boundary because "Python has no private constructors" —
`__new__`, `copy`, `dataclasses.replace` and pickle all mint an
`AuthorizedAction` with no gate involvement. That retraction is
language-specific and it partially reverses in JavaScript.

```text
class AuthorizedAction { #brand; ... }

Object.create(AuthorizedAction.prototype)   -> brand check fails
structuredClone(authorized)                 -> private fields not copied
JSON round-trip                             -> brand check fails
```

An ECMAScript private field is a genuine runtime brand, not a lint. It
defeats exactly the class ADR-002 D2's HMAC token was introduced to
defeat — model-authored code fabricating an authorisation object — at
zero cost and with no key to protect.

**Ceiling, stated rather than talked around:** any code in the same
realm that can reach the gate's constructor, or monkey-patch the module,
bypasses it. This is not a trust boundary. It replaces tier 2, not
tier 3, and ADR-003's property still needs a real backend.

---

## 3. What Core decided that GRU contradicts or has already superseded

### 3.1 ADR-001's ordering invariant is dead for GRU

**FACT.** ADR-001: "Effect Gate exists before Executor exists before
Model can propose executable actions. No commit may introduce an
ungoverned model-controlled execution path." **FACT.** GRU §21 adopts
DeepSeek Harness precisely *because* it already provides "agent loop and
tool execution", "terminals and local filesystem capabilities" and
"persistent workspace records".

**INFERENCE.** GRU cannot satisfy ADR-001 as written. The executor is
the foundation, adopted on day one; the gate arrives afterwards. This is
the pattern ADR-001 describes — "most agent frameworks grow an execution
path first and retrofit safety around it" — chosen deliberately.

But ADR-001's premise was already withdrawn *inside Core*. The effect
surface audit found `run_check.py --shell` came across in the clone and
concluded: "Core *does* inherit an ungoverned path. The ordering
invariant is still the right rule; the claim that Core starts from a
clean slate is withdrawn."

```text
WHAT DIES     the ordering rule. There is no commit sequence in which
              GRU's gate precedes GRU's executor.

WHAT SURVIVES the property the ordering was protecting: no consequential
              effect may reach the environment by a route AVL does not
              mediate. That is a CONFORMANCE obligation, not a build
              order, and GRU §32.2 already has the slot for it.
```

**RECOMMENDATION.** ADR-001 does not carry forward. Its replacement is a
route-bypass conformance matrix under GRU §32.2, built with the effect
surface audit method. Carrying ADR-001 into GRU unchanged would create a
rule the project violates on its first commit, and a violated invariant
teaches the team that invariants are aspirational.

### 3.2 ADR-003's process separation has no GRU counterpart, and GRU's posture contradicts its fail-closed rule

**FACT.** ADR-003 §6: "If no available backend satisfies the property,
Core refuses untrusted autonomous execution or enters an explicitly
labelled unsafe development mode. It does not proceed quietly on a
weaker boundary."

**FACT.** GRU §20 lists OpenShell as a "STRONG OPTIONAL BACKEND" and
Cedar as a "CANDIDATE DEPENDENCY". **FACT.** GRU §31.1's v0.1 scope
includes no sandbox, no isolation backend and no policy engine —
§31.2 explicitly defers "complex formal policy engine before AVL v0
proves the required semantics".

**INFERENCE.** GRU v0.1 as scoped fails P1, P2 and P8 by construction.
Minions run with whatever ambient authority the Node process has;
nothing confines them; AVL v0 is a library in the same process as the
code it governs. That is the same position the threat model records for
v1 — "five FAIL, five N/A, zero PASS" — reached by a different route.

This is not an argument against GRU v0.1. Deferring isolation to prove
the organisation thesis first is a defensible sequencing choice. The
contradiction is that Core requires the weaker mode to be *labelled* and
GRU does not say it is in one.

**RECOMMENDATION.** GRU adopts one sentence: until an isolation backend
satisfying P1/P2/P8 is admitted, GRU runs in an explicitly labelled
unsafe development mode and AVL reports it. Cost: one status field.

### 3.3 "Governance precedes execution" is superseded by "governance is conformance-tested against execution"

Covered by §3.1. Stated separately because the slogan appears in the
Core README as the repository's invariant and will otherwise survive by
inertia.

### 3.4 ADR-002 D1's "three of six already earned" does not survive

**FACT.** ADR-002 D1: `L`, `V`, `T` are "already earned by v1, migrated
not reinvented". **FACT.** GRU is TypeScript on a Node chassis; v1 is
Python.

**INFERENCE.** Under GRU, all six are specification. The chassis
supplies part of `L`'s plumbing (sessions, workspaces, replay-oriented
state per §21) and GRU's own Project Contract and Organization Compiler
supply the rest. Core's scope statement — reproduced in the README as
`INHERITED / NEW / IMPORTED` — is inaccurate the moment the target
language changes.

### 3.5 ADR-004's control is stale, which makes EXPERIMENT_CORE_000 unrunnable as written

**FACT.** ADR-004's strong null is "v1 delivery semantics + Claude Code
or Codex as executor + host governance/AGT + CI + v1 evidence closure +
normalized episode record + offline improvement proposal + external
promotion gate". **FACT.** ADR-004 rejects the alternative "skip the
spike": "Reasoning that has been wrong once at this exact level of
confidence does not get to substitute for a test the second time."

**INFERENCE.** GRU is a fourth term the control does not contain: a
governed delivery organisation with a constitutional layer, running on a
chassis that already provides sessions, tools, routing and a web client.
ADR-004's own logic — specify the *strongest* fair comparator — requires
that comparator to include GRU. The preregistered predictions were
written against a landscape that no longer holds.

An experiment whose control is stale and whose subject (a Core-owned
Python executor) has no sponsor will not be run. Leaving it marked
`PREREGISTERED -- not yet run` implies it is pending. It is not.

### 3.6 ADR-005's three-line taxonomy loses its middle term

**FACT.** ADR-005: "v1 proves the harness / Core industrialises the
trust boundary / Continuum researches what Core still cannot solve."

**INFERENCE.** GRU takes the middle line's job. Either the taxonomy
gains a fourth term or Core's line is vacated. It cannot stay as written.

### 3.7 A small one, cheap to fix now

**FACT.** Gap analysis "Naming and repository structure": Core was
created by cloning v1 with "deliberately **no** GitHub fork
relationship: PRs must not default upstream, and the repository must be
freely private-able." **FACT.** GRU §21.2 contemplates deriving from
DeepSeek Harness history with `upstream → deepseek-ai/deepseek-harness`.

**INFERENCE.** If GRU is derived as a GitHub fork, it reintroduces the
default-upstream-PR problem and the private-ability problem Core solved
by clone-and-push. GRU §21.2's stated intent (selective adoption, not
blind merge) is satisfied by a remote, not by a fork relationship.

---

## 4. Four candidate relationships

### (a) GRU supersedes Core entirely

**Strongest case for.** GRU §2 states it outright: "Escapement is no
longer the mandatory technical foundation," and §0: "GRU does not
inherit their implementation or architecture by default." Core is
design-only, frozen, gated on an experiment whose control is stale, and
three of its five ADRs are contradicted, weakened or superseded by a
chassis that already executes. GRU v1.1 is frozen with its own
constitutional layer named, scoped and assigned BUILD NATIVE. §35
forbids keeping our own work for being ours, and this project has
already lost a year to that bias.

**Strongest case against.** "Supersede" asserts that GRU covers what
Core covered. It does not. GRU §14 is a thirty-line list of
responsibilities; it has no adversary, no properties, no episode schema,
no eligibility predicate, no verification ladder and no conformance
method. GRU §32.1 item 4 asks Core's central question and supplies no
way to answer it. GRU does not supersede Core's content — it has a hole
the exact shape of it.

**What would settle it.** Write GRU's AVL conformance experiment from a
blank page. If it reaches the quality of the threat model plus the
route-bypass matrix in under a week, Core's corpus has no future use and
(a) is correct.

### (b) Core becomes GRU's AVL specification

**Strongest case for.** The mapping is near-exact at the semantic layer
(§1.1). AVL's owned list — ProjectAction semantics, ProjectConsequence
normalization, AuthorityRequirement, EvidenceRequirement,
VerificationContract, state-transition eligibility, truthful closure,
DirectorOverride, learning and promotion eligibility — is `K` + `E` + `P`
almost line for line. GRU §17.1 and §20 both assign those semantics to
BUILD NATIVE, and §34 freezes "AVL as GRU constitutional layer" while
leaving the policy plumbing provisional. That is precisely the
`K_semantics` / `K_enforcement` split, reached independently by two
documents that never saw each other. Independent convergence is evidence
the content is real rather than ours. And the transfer is free: prose
and JSON, no migration.

**Strongest case against.** Core's corpus was written for a system whose
central premise — Escapement owns the mandatory mediation point — was
never validated and is exactly what EXPERIMENT_CORE_000 existed to test.
Adopting it imports an unvalidated premise into a frozen architecture
with the validation now permanently deferred. The one empirical result
Core ever produced was that its own founding claim was false. A
specification whose founding question was never answered is not
obviously safer than a blank page; it is a blank page with four rounds
of confident prose attached.

**What would settle it.** Run GRU §32.1 item 4 twice: once with the
threat model and route-bypass matrix as the instrument, once without.
If the instrumented run finds no route the uninstrumented run missed,
the corpus is decoration.

### (c) Core continues as an independent Python line alongside GRU

**Strongest case for.** ADR-005's taxonomy says v1, Core and Continuum
are for different things, and v1 is a real shipped Python product — 296
files, 189 tests, 122 routing evals, a doctor, a security gate. Core is
v1's trust layer. GRU is a new product in a new language at v0.1 scope
with an alpha-status chassis carrying "compatibility-breaking-change
warnings". Killing the mature line for the immature one is a bet, and
ADR-005 D3 already sequenced v1 hardening *ahead* of Core for exactly
that reason.

**Strongest case against.** Core is the part of that line with zero
implementation. Continuing it means one operator building two
constitutional layers in two languages — duplicating the single hardest
and least-parallelisable piece of work in either project, at the exact
moment the hard part is the conceptual one that transfers for free.
Nothing in (c) is achieved that (b) does not achieve, and (b) costs one
programme instead of two. v1 hardening is unaffected either way: it is
v1 work, not Core work.

**What would settle it.** Whether v1 acquires users who need a trust
layer in Python on a schedule GRU cannot meet. That is a commercial
fact, not an architectural one, and no document here contains it.

### (d) Core is archived as a research record

**Strongest case for.** It is design-only, frozen, superseded in its
execution-owning conclusions, and its ADR chain records a build
programme nobody will build. Archiving is honest, cheap and reversible.
Prefer subtraction.

**Strongest case against.** Archiving is only correct if the corpus has
no future use, and §2 lists eight artifacts GRU demonstrably lacks and
cannot get for free anywhere else. Archiving them is not subtraction —
it is deleting the answer and keeping the question. And GRU §35's rule
cuts both ways: "Do not replace strong infrastructure merely to claim
GRU built everything."

**What would settle it.** The same test as (a).

---

## 5. Recommendation

**RECOMMENDATION.** (b), bounded, with (a) applied to everything (b)
does not name.

```text
GRU ABSORBS      Core's constitutional semantics, threat model,
                 episode schema and validation method, as input to AVL

GRU SUPERSEDES   Core's build programme: executor, kernel/worker
                 topology, IPC, broker implementation, Python migration
                 sequence, LangGraph seam, the ordering invariant

CORE SHAPES GRU  at exactly one layer -- AVL acquires an adversary, ten
                 properties, an episode record and a conformance test
                 it does not currently have

NOTHING RUNS     there is no case for two constitutional layers in two
ALONGSIDE        languages maintained by one operator
```

The reasoning, in order:

1. Core has no code, so GRU §35's rule does not apply to it. The rule
   applies to v1, and v1's inheritance is what the language change
   actually destroys (§0).
2. The seven Core artifacts in §2 are prose and JSON. Their transfer
   cost is reading time.
3. GRU has a named, frozen, BUILD-NATIVE slot for exactly this content
   (§17.1, §20, §34), and it is currently thirty lines long.
4. GRU asks Core's central question in §32.1 item 4 with no method
   attached. Core's method is the only one either project has.
5. Everything Core decided *about building an executor* is moot under a
   chassis that already executes (§3.1–§3.5). It goes.

**Concretely, and each of these is a subtraction:**

```text
CLOSE, not freeze     ADR-004 and ADR-005 are amended to record that the
                      Core build programme is closed and the experiment
                      will not run as preregistered. A frozen programme
                      with a stale control reads as resumable and will
                      attract sunk-cost effort.

MARK DEAD             gap analysis §B (v1 reuse map), §D (scripts/core
                      extraction), §I (LangGraph seam), §L (18-step
                      migration); ADR-001's ordering rule; ADR-003's
                      process topology; ADR-002 D1's inheritance claim.
                      Dead means annotated in place, not deleted --
                      the retraction convention is itself an asset.

REISSUE               EXPERIMENT_CORE_000 as GRU's AVL conformance
                      experiment: same invariants S1-S8, same threat
                      model, same reduction ladder, new control (GRU on
                      DeepSeek Harness), new predictions written before
                      the run.

CARRY FORWARD         the eight artifacts in §2, minus Zones A/B/C,
                      which are reserved until GRU has a self-
                      modification runtime to bound.
```

**What would change my mind.**

```text
A blank-page AVL conformance experiment reaching the same quality in a
week   -> (a). The corpus is decoration and should be archived.

GRU abandoning the Node chassis and returning to Python
       -> (c) reopens on inheritance grounds alone.

v1 acquiring users who need the trust layer in Python on a schedule
GRU cannot meet
       -> (c) on commercial grounds, regardless of architecture.

The instrumented-vs-uninstrumented run in §4(b) finding no extra route
       -> the threat model is not load-bearing; keep the episode schema
          and drop the rest.
```

**What would not change my mind:** the amount of work in `docs/core/`.
Four rounds of independent review is a reason to trust the *content*, not
a reason to keep the *programme*. Those are different propositions, and
conflating them is the failure mode ADR-002 already caught once.

---

## 6. Open questions the documents do not answer

```text
1  Is AVL a boundary or a library?
   GRU §5 draws it as a layer. Nothing says it is a separate process,
   principal or realm. Core presumed separation (ADR-003) and GRU is
   silent. Until answered, P1/P2 are unevaluable for GRU.

2  Who authorises Nefario's ADMIT?
   GRU §28 says installation does not imply authorization and names no
   authoriser. Director, AVL, or Nefario itself are three very
   different systems.

3  What is a GRU episode?
   §13 lists "Verified Episodes" as Minion Mind contents with no
   schema. Core's schema, a new one, or the chassis's session records?

4  Does a Director override make an episode learning-eligible?
   Closure FAIL + Director PROCEED + residual risk accepted is
   truthful under GRU §7 and has no verdict under Core schema §5.2.
   Neither document has met this case; it exists only once the two
   are combined.

5  Who writes AVL's conformance tests?
   Core's standing rule is that self-authored verification fails
   systematically, and schema §10 Q3 notes the author is now one model
   rather than one person, "which is not obviously better".

6  Does EXPERIMENT_CORE_000 ever run, and against what?
   §3.5 argues not as written. Nothing states the replacement.

7  Licence path.
   v1 is Apache-2.0; Core is source-available and "may relicense once
   Experiment 000 has run" -- an event §3.5 argues will not occur;
   DeepSeek Harness is MIT. What licence the transferred
   specifications carry into GRU is undecided.

8  Is Escapement v1 still a maintained product line?
   ADR-005 D3 sequences v1 hardening first. GRU does not mention v1 as
   a maintained line at all -- only as research lineage.

9  Multi-project and multi-tenant.
   Core's threat model excludes both. GRU v0.1 is one local project
   while the product thesis implies many. The trust classes will need
   revision before that transition, not after.

10 Prompt injection.
   T4 is acknowledged and unmodelled in Core §6 and unmentioned in
   GRU. GRU's exposure is larger: Nefario reads repositories, APIs,
   docs and papers to acquire capability, which is a T4 surface
   feeding a T3 component that performs T2 admission.
```

---

## Related

- `docs/decisions/ADR-001` .. `ADR-005`
- `docs/core/CORE_THREAT_MODEL_v0.1.md` — the primary transfer
- `docs/core/EXPERIMENT_CORE_000_PREREGISTRATION.md` — the method to reissue
- `docs/core/EFFECT_SURFACE_AUDIT.md` — the method GRU needs most
