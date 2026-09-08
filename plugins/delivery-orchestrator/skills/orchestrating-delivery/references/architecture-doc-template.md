# Architecture Doc Template

Use this when scaffolding a repo's architecture set (documentation-pass Step 1)
or when structuring a new reference doc or ADR. The architecture set is
**skill-shaped**: a concise main doc that stays readable end-to-end, plus
per-subsystem detail one click away, plus an append-only decision log.

## `architecture/architecture.md` (the main doc)

```markdown
# <System> — Architecture

> Living architecture. Updated as a development stage via the documentation pass
> (see <design-home>/README.md). Reflects the system as built **today**.

## Overview

2-4 paragraphs: what the system is, its primary responsibility, the runtime it
lives in, and the handful of load-bearing ideas someone must hold in their head
before any detail makes sense. No changelog, no history — present tense.

## Core building blocks

The basics that change rarely and that every reference assumes. Keep each to a
few sentences; push detail into a reference. Typical entries (adapt to the
system): runtime/execution model · identity & auth · storage & tenant/env
isolation · the core domain abstraction · external interfaces (APIs/UI/MCP) ·
cross-cutting concerns (scheduling, observability).

## Reference manifest

One row per detail doc. The hook is a single sentence so a reader can decide
whether to open it. Keep this table in sync with `references/` (every file
listed; every row resolves).

| Subsystem | Reference | What it covers |
|---|---|---|
| <name> | [references/<file>.md](references/<file>.md) | <one-line hook> |

## Decisions

Why it's built this way lives in the append-only log: [decisions/](decisions/).
```

## `architecture/references/<subsystem>.md` (a detail doc)

```markdown
# <Subsystem>

## Responsibility
One paragraph: what this subsystem owns and what it deliberately does not.

## Structure
The components/modules and how they fit. A small inline Mermaid diagram is fine
when a picture genuinely helps; don't force one.

## Contracts
Public surfaces other parts depend on — APIs, tool signatures, data/store shapes,
message formats. Paste the real shapes; these are load-bearing.

## Lifecycle / flow
The control or data flow through this subsystem, in present tense.

## Constraints & decisions
Non-obvious rules and invariants. Link the ADRs in `../decisions/` that
established the material choices.
```

## ADR / decision-record format

The decision log is **append-only**: records are never deleted or rewritten away;
a reversed choice gets a new ADR and the old one's status flips to `superseded`.
The prose docs say how things work *now*; the log says *why*. Record a decision
only once it is **implemented** (shipped).

`architecture/decisions/README.md` (the index + format):

```markdown
# Decisions (ADR log)

Append-only record of material, **implemented** architecture decisions — the
"why" behind non-obvious choices. Present-tense "how it works" lives in
[../architecture.md](../architecture.md) and the references; this is history.

One file per decision: `NNNN-<slug>.md` (zero-padded, monotonic). Reversing a
decision = a new ADR + flipping the old one's status to `superseded by NNNN`.

| # | Decision | Status |
|---|----------|--------|
| 0001 | <title> | accepted |
```

Each `architecture/decisions/NNNN-<slug>.md`:

```markdown
# NNNN — <short title>

- **Status:** accepted | superseded by NNNN | supersedes NNNN
- **Date:** YYYY-MM-DD
- **Source:** <link to the plan/spec that implemented this>

## Context
The forces and the obvious alternative(s) — why a choice was even needed.

## Decision
What was chosen, stated plainly.

## Consequences
What this buys, what it costs, and any follow-on constraints it imposes.
```

## Granularity

One reference **per subsystem**, not per plan and not per individual skill/feature.
A subsystem is a coherent area of responsibility (auth, storage, the scheduling
subsystem, the gateway). Create a new reference only when a genuinely new
subsystem appears; otherwise extend the existing one so the picture stays whole.
ADRs, by contrast, are **per decision** — one shipped material choice each.
