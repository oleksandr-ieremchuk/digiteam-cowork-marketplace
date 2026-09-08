# 0002 — Two plugins, split by tool

- **Status:** accepted
- **Date:** 2026-09-08
- **Source:** `design_docs/marketplace-bootstrap-design.md` (D2)

## Context
The delivery process has two halves with opposite rules: the Cowork half must
never run git or edit code; the Claude Code half must move plan stages with
`git mv` and commit. Plugin format is identical across the two products, so a
single plugin holding both skills would install and work in either. The
question was whether to ship one plugin or two.

## Decision
Two plugins: `delivery-orchestrator` (install into Cowork, skill
`orchestrating-delivery`) and `delivery-executor` (install into Claude Code,
skill `executing-delivery-handoff`).

## Consequences
Each tool loads only the half meant for it — the "never touch git" rules never
sit in a Claude Code context, and the executor's git procedure never sits in
Cowork, so neither can mis-trigger. Cost: the HandOff and Report shapes must be
kept in both plugins (see ADR 0004's parity rule), and users install two things
instead of one.
