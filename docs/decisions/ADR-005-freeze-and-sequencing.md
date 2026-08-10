# ADR-005 — Core design freeze; v1-first sequencing

```text
STATUS    Accepted
DATE      2026-08-10
EFFECT    Core design work stops here. Engineering attention moves to
          v1. Core resumes at EXPERIMENT_CORE_000.
```

## Context

Four rounds of independent review took Core from a ten-item ownership
list to a preregistered experiment that may conclude Core should own
almost nothing. The last round falsified the founding claim. The design
is now coherent enough to freeze and too unproven to build.

Meanwhile the reviews' actual message is that Escapement needs a
stronger mechanical trust layer before more orchestration intelligence —
which v1 can begin delivering without waiting for Core.

## Decisions

**D1 — Core is frozen at this commit.** No executor, kernel, broker, IPC
or MCP surface. It resumes when EXPERIMENT_CORE_000 runs. Everything
needed to resume is in `docs/core/` and `docs/decisions/`; nothing is
carried in a person's head.

**D2 — Continuum stays frozen.** It reopens only when Core meets a
concrete problem that deterministic mechanisms handle badly. A research
line reopened on enthusiasm rather than a blocked problem is how Phase I
consumed a year.

**D3 — Sequencing.**

```text
1  open-source cleanup      licence decision, provenance audit,
                            contribution docs, maturity messaging
2  v1 hardening             code intelligence, independent verification,
                            deterministic evidence, honest claims
3  v1 MCP v0.1              thin, read/governance oriented
4  Core implementation      gated on EXPERIMENT_CORE_000
5  Core Agent + MCP         one kernel, two interfaces
6  Continuum                frozen
```

**D4 — An agent is not an MCP server.** MCP is a communication surface;
an agent is a thing that acts. Core will eventually expose *both*
interfaces over one kernel:

```text
        Agent API    MCP    CLI
             \        |     /
            trusted governance
            effect boundaries
            verification, evidence
```

Saying "Core becomes an Agent MCP" collapses the two and buys a cleanup
later. The wording is fixed now, while it costs nothing.

**D5 — v1 MCP and Core MCP do not talk to each other as peers.** During
migration each fronts its own runtime. Core may later expose a v1
compatibility adapter so it can *consume* v1 functionality — but v1 is a
library to Core, never a second intelligent agent it negotiates with.
Two autonomous MCP servers in conversation is a distributed system
nobody asked for.

**D6 — Do not make Continuum a product surface.** Turning a research
architecture into a product interface before its mechanism has earned a
place is precisely the error Phase I ended on.

## The line that survives all of this

```text
v1         proves the harness
Core       industrialises the trust boundary
Continuum  researches what Core still cannot solve
```

Better than "v1, then v2, then v3", because it says what each is *for*
rather than implying each supersedes the last.

## Consequences

Two items leave Core's scope and become v1 work, both improved by the
move: `run_check.py --shell` is a v1 security fix with its own test, and
the effect surface audit should become a v1 `doctor` check so the claim
is re-verified per commit rather than asserted once in prose.

`docs/core/` is design-only at this freeze — no Core implementation
exists, and the inherited v1 runtime is untouched.
