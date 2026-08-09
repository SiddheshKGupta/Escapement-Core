# Effect Surface Audit — v1 at Core's fork point

```text
STATUS    complete for EXECUTION and NETWORK; partial for FILESYSTEM
VERDICT   the founding claim was FALSE as stated
SCOPE     scripts/*.py, 14 files, at Core commit fec3c39
```

## The claim under audit

Gap analysis §A, ADR-001 and the review handoff all assert:

> v1 has no model-to-effect path at all.

Evidence offered:

```bash
grep -rniE "subprocess|os\.system|exec\(" scripts/agent_runtime.py scripts/capability_router.py
# no matches
```

**That evidence was insufficient and the conclusion it supported is
false.** The grep covered 3 patterns across 2 of 14 files. Widening it
to the standard effect primitives across all of `scripts/` returns **47
matches in 7 files**. Absence of three strings in two files was
presented as absence of an effect surface.

This is the A1 vacuity failure from our own contract clauses — a check
that passes because it cannot fail — committed in the document that
establishes Core's founding property, and repeated in three artifacts
before an external reviewer caught it.

## What is actually there

### EXECUTION — a general-purpose path exists

```text
scripts/run_check.py
  --shell "<arbitrary string>"   -> subprocess.run(..., shell=True)
  -- <arbitrary argv>            -> subprocess.run(argv)
```

One `shell=True` site exists in the repository and it is reachable from
the command line with a caller-supplied string. The argument's help text
reads:

> "Run one trusted shell command string instead of argv after --."

Nothing establishes, checks or records that the string is trusted. The
word is the entire control.

```text
scripts/codex_resources.py:360
  query_resource_state(command: Sequence[str]) -> subprocess.Popen(list(command), ...)
```

Caller-supplied command sequence, no allowlist.

The remaining execution sites are benign by construction: 15 uses of
`sys.executable` with a literal script path, which is CLI subcommand
dispatch (`escapement.py doctor` re-invoking a sibling script), plus one
`["git", "ls-files"]`. Those are fixed argv with no interpolation and are
not an effect surface in the relevant sense.

### NETWORK — clean

No egress primitives. The only `urllib` import is `urllib.parse` in
`local_viewer.py` for parsing, not fetching. This part of the original
claim holds.

### FILESYSTEM — present everywhere, audit incomplete

Write, delete and mkdir primitives appear in 10 of 14 files, most
heavily in `agent_runtime.py` (15) and `escapement.py` (13). This is
expected — the runtime persists JSON records — but **no path-confinement
analysis has been done**, and "the tool writes where it is told" is not
the same as "the tool cannot be told to write anywhere." Marked
incomplete rather than passed.

## The honest reformulation

The interesting question is not whether an execution primitive exists —
it does — but whether v1 *grants* the model capability it did not
already have.

It does not. v1 runs inside hosts (Claude Code, Codex) where the agent
already has a Bash tool. `run_check.py --shell` gives the model nothing
it could not do directly; it is a **recording wrapper** over execution
the host already permits. Its purpose is evidence, not authority.

So the defensible statement, replacing the false one:

```text
FALSE   v1 has no model-to-effect path.

TRUE    v1 adds no execution capability beyond what its host already
        grants the model. Its one general-purpose runner exists to
        record evidence, not to confer authority, and its governance
        over that runner is advisory: nothing forces the model to
        route an effect through it rather than around it.
```

That is a materially weaker position than the one Core was founded on,
and it must be stated in those terms.

## Consequences

**ADR-001's rationale is damaged; its decision is not.** The ADR argues
that most frameworks grow an execution path first and retrofit safety,
leaving an ungoverned window, and that "Core does not have to inherit
that." Core *does* inherit an ungoverned path — `run_check.py --shell`
came across in the clone. The ordering invariant is still the right
rule; the claim that Core starts from a clean slate is withdrawn.

**Core's differentiator shifts.** Not "governance existed before
execution" — execution was already there. Rather: **Core is the point at
which the recording wrapper stops being optional.** v1 records effects
that happen to be routed through it. Core's claim is that no
consequential effect can be routed around it. That is a real difference
and it is still worth building toward, but it is a claim about
mandatoriness, not about a clean start.

**One concrete remediation, independent of Core.** `--shell` is a
`shell=True` site whose only safeguard is the word "trusted" in a help
string. Whether or not Core is built, that argument should either be
removed or gated. It is the single highest-risk line in v1 and it is in
the file whose job is producing trustworthy evidence.

## What this audit does not cover

```text
filesystem path confinement          not analysed
the 35 native skills                 not analysed
external capability resources        host-dependent by design
MCP / plugin dispatch surfaces       do not exist in v1 yet
transitive imports outside scripts/  not traced
```

An audit that stops where the auditor stopped looking is how this
document's subject came to be written in the first place. The
uncovered areas are listed so the next reader knows the boundary rather
than inferring a clean result from silence.

## Method, for repetition

```bash
grep -rnE "subprocess|os\.system|os\.popen|os\.exec|os\.spawn|\beval\(|\bexec\(|\bcompile\(|__import__|importlib|ctypes" scripts/ --include=*.py
grep -rnE "\b(requests|urllib|httpx|http\.client|socket|urlopen)\b" scripts/ --include=*.py
grep -rnoE "write_text|write_bytes|shutil\.(rmtree|copy|move)|os\.remove|\.unlink\(|os\.makedirs|mkdir\(" scripts/ --include=*.py
grep -rn "shell=" scripts/*.py
```

This should become a doctor check, so the claim is re-verified on every
commit rather than asserted once in prose.
