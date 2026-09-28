# Architecture Doc Template

Use this when scaffolding a repo's architecture set (documentation-pass Step 1)
or when structuring a new reference doc or ADR. The architecture set is
**skill-shaped**: a concise main doc that stays readable end-to-end, plus
per-subsystem detail one click away, plus an append-only decision log. The
same file holds the per-repo spec template and the feature-record template.

Everything here is repo-local: it describes only this repo. Other repos and
systems appear only by name, as a contract counterparty, with the shape written
out. Never a path or link into another repo.

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

## Contracts & integrations

What this repo offers others and what it depends on. Counterparties are named,
never linked into another repo; the shape is written out here or in a file in
this repo.

### Provides

| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |
|---|---|---|---|---|---|
| <who uses it> | <API / plugin / file / message> | <inline, or a path in this repo> | <how callers authenticate, or none> | <error shape and failure modes> | <how it is versioned> |

### Consumes

| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |
|---|---|---|---|---|---|
| <what this repo depends on> | <API / plugin / file / message> | <inline, or a path in this repo> | <credential this repo uses, or none> | <how this repo handles failure> | <version it pins or expects> |

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
Split as in `architecture.md`, with the same columns (counterparty,
kind/protocol, shape, auth, errors, versioning):
- **Provides** — public surfaces others depend on: APIs, tool signatures,
  data/store shapes, message formats.
- **Consumes** — what this subsystem depends on, inside or outside this repo.
Paste the real shapes; these are load-bearing.

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
- **Source:** <path in this repo to the plan or spec that implemented this>

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

## Per-repo spec — `design_docs/<slug>-design.md`

Every target repo gets its own spec, describing only the changes to make in
that repo. It follows the repo-local rule: other repos appear by name, as the
counterparty of a contract, with the shape written out.

```markdown
# <Title> — v<X.Y.Z>

- **Slug / Status / Author / Release**

## 1. Problem
As it shows up in this repo.

## 2. Decisions
| # | Decision | Alternative rejected | Why |
|---|---|---|---|

## 3. Changes in this repo
What changes, file by file or area by area. End with **Out of scope**.

## 4. Contracts
**Provides** and **Consumes**, each a table with the shape written out
(columns as in `architecture.md` Contracts & integrations).

## 5. Acceptance criteria
Numbered, each checkable in this repo.

## 6. Integration verification
Single-repo feature only. For a multi-repo feature it lives in
`<slug>-feature.md`.
```

## Feature record — `design_docs/<slug>-feature.md` (primary repo only)

Only for a multi-repo feature, and only in its primary repo. This is the one
file allowed to talk about several repos. It is not folded into the primary
repo's architecture docs, and no per-repo spec links to it. A single-repo
feature has no feature record: its `<slug>-design.md` doubles as one.

```markdown
# <Title> — feature record — v<X.Y.Z>

- **Slug / Status / Author / Release / Primary repo**

## Goal
One paragraph: what the feature achieves end to end.

## Target repos
One line each: repo name and its role in the feature.

## Per-repo scope
One short paragraph per repo; the detail lives in that repo's own spec.

## Contract matrix
| Producer | Consumer | Contract | Shape |
|---|---|---|---|

Each row must appear, with the identical shape, in the producer's Provides and
the consumer's Consumes (gate "specs consistent").

## Shared decisions
Decisions that bind more than one repo.
| # | Decision | Alternative rejected | Why |
|---|---|---|---|

## Integration-verification plan
What phase 5 exercises, per contract-matrix row.
```
