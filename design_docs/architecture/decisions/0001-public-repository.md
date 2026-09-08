# 0001 — Public repository

- **Status:** accepted
- **Date:** 2026-09-08
- **Source:** `design_docs/marketplace-bootstrap-design.md` (D1), plan `marketplace-bootstrap`

## Context
The marketplace must be installable from both Claude Cowork and Claude Code.
Claude Code can add a marketplace from a private GitHub repo through git
credentials; Cowork's "Add marketplace" accepts only public GitHub repositories.
The content is process description — nothing confidential.

## Decision
The repository `oleksandr-ieremchuk/digiteam-cowork-marketplace` is public,
MIT-licensed, and serves both products from one URL.

## Consequences
One repo, one install path per product, no credential setup for Cowork. Anything
committed here is world-readable, so personal or company-specific detail stays
out of the skill text (see the generic-wording rule in the spec, D3).
