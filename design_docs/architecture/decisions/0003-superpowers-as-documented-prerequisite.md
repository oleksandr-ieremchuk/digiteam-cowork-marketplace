# 0003 — superpowers as a documented prerequisite, not a declared dependency

- **Status:** accepted
- **Date:** 2026-09-08
- **Source:** `design_docs/marketplace-bootstrap-design.md` (D4)

## Context
Both skills delegate to superpowers: the orchestrator calls
`superpowers:brainstorming`, `superpowers:writing-plans` and
`superpowers:executing-plans`; the executor calls `superpowers:writing-plans`
and `superpowers:executing-plans`. Plugin manifests support a `dependencies`
field, including cross-marketplace dependencies, but those need an
`allowCrossMarketplaceDependenciesOn` allowlist and break when the third-party
marketplace or plugin is renamed. superpowers lives in a marketplace this repo
does not control.

## Decision
No `dependencies` entry. The README (root and per-plugin) states the
prerequisite and gives the install commands.

## Consequences
Installing either plugin never fails on dependency resolution; the failure mode
moves to runtime — a skill call to a missing `superpowers:*` skill — which the
README is meant to prevent. If superpowers ever ships in a first-party
marketplace, revisit this with a new ADR.
