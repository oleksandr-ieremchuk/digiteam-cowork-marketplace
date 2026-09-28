# 0009 — Gate "specs consistent" closes phase 0a

- **Status:** accepted
- **Date:** 2026-09-28
- **Source:** `design_docs/repo-local-docs-design.md` (D6, D12); plan
  `repo-local-docs`, released as `v0.1.2`

## Context
With one spec per repo (ADR 0008), each side of a cross-repo contract carries
its own copy of the shape. Without a check, the copies could disagree and the
mismatch would surface only at phase 5, after code is written and deployed.

## Decision
Phase 0a ends with a Cowork-owned gate, **specs consistent**, before any 0b
HandOff: every row of the feature record's contract matrix appears, with the
identical shape, in the producer's Provides and the consumer's Consumes, and
every per-repo spec is self-contained. For a single-repo feature only the
self-containment check applies. A failure is fixed in the specs; no HandOff goes
out until the gate passes. Gate 1 checks each plan against its own repo's spec
only, and phase 5 derives the contracts from the matrix plus each repo's
Provides / Consumes.

## Consequences
Contract drift is caught on paper, before planning. Phase numbering is
unchanged: the gate is attached to 0a rather than added as a new phase.
