# 0004 — Chat-only HandOff / Report protocol, no shared state

- **Status:** accepted
- **Date:** 2026-09-08
- **Source:** the `orchestrating-delivery` skill design (pre-dating this repo), shipped here as both plugins' `references/`

## Context
Cowork and Claude Code share no runtime, no plugin store and no process
database. The Cowork sandbox cannot run git reliably. The two halves still have
to agree on where a feature is and what happens next.

## Decision
The only interface is two copy-paste chat blocks — a HandOff (Cowork → Code, one
per repo) and a Report (Code → Cowork) — and they are never written to files.
Durable process state is the plan's stage folder in the target repo
(`drafts/ongoing/done/documented`), moved only by Code. Both plugins carry their
own copy of the block shapes, and the Report block must stay byte-identical
between them; the orchestrator's integration gate checks this.

## Consequences
The process is resumable from disk at any time and survives either tool losing
context; no infrastructure to run. The human carries the blocks between tools,
which is one manual step per phase. Redundant copies of the shapes are a
deliberate maintenance cost — any change to a block is a change to both plugins
in the same release.
