# 0008 — Repo-local specs and docs; the spec is the scope

- **Status:** accepted
- **Date:** 2026-09-28
- **Source:** `design_docs/repo-local-docs-design.md` (D1–D5, D7–D9); plan
  `repo-local-docs`, released as `v0.1.2`

## Context
In use, the documentation pass produced feature-level docs and linked one
repo's docs into another's. The skill ran one pass per feature, fed it the
single cross-repo spec that every repo's HandOff pointed at, allowed ADR
`Source:` to name another repo, and gave consumed contracts no home. The obvious
alternatives were to keep one shared spec and just forbid cross-links, or to
keep the feature picture in a tool outside the repos (an issue tracker or the
Cowork project); the first leaves the leakage's root cause in place, the second
fails wherever that tool is absent.

## Decision
- Everything written into a repo — spec, plan, architecture docs, ADRs —
  describes only that repo. Other repos and systems appear only by name, as the
  counterparty of a contract, with the shape written out. No path or link into
  another repo.
- Every target repo has its own `design_docs/<slug>-design.md`. A multi-repo
  feature also has `design_docs/<slug>-feature.md` in a primary repo (proposed
  by Cowork, confirmed by the user): the only file allowed to talk about several
  repos. A single-repo feature has only the design file, which doubles as the
  feature record. Cowork discovers features by these files and asks when that
  is ambiguous.
- The HandOff's `Spec` is this repo's own spec and is the whole scope; the
  `Your scope in this repo` field is removed. No primary-repo field is added,
  because Code never needs another repo. The Report is unchanged.
- `architecture.md` has a mandatory **Contracts & integrations** section with
  Provides and Consumes tables (counterparty, kind/protocol, shape, auth,
  errors, versioning). The documentation pass runs once per repo from that
  repo's spec, plan and code; ADR `Source:` names a path in the same repo.

## Consequences
Each repo's docs are readable and true on their own. Contract shapes are
duplicated on both sides of a contract by design; the "specs consistent" gate
(ADR 0009) and phase 5 are what keep the copies in agreement. The executor now
blocks on any path into another repository instead of following it.
