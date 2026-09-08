# 0005 — Naming, versioning and author fields

- **Status:** accepted
- **Date:** 2026-09-08
- **Source:** `design_docs/marketplace-bootstrap-design.md` (D5, §3.3, §3.5); plan `marketplace-bootstrap` (DP1, DP2, DP3)

## Context
The marketplace name is the suffix users type on every install; the two plugins
ship together; `claude plugin tag` produces `<plugin>--v<version>` tags, one per
plugin; the spec's generic-wording check (`grep -ri chatrevenue plugins/` must
be empty) conflicted with putting the owner's email in `plugin.json`; and the
validator rejects a `$schema` key in `marketplace.json`.

## Decision
- Marketplace `name` is `digiteam` (not the repo name), so installs read
  `delivery-executor@digiteam`.
- Both plugins and `metadata.version` share one version; releases are one
  annotated repo-level tag `vX.Y.Z` on the verified commit.
- `plugin.json` `author` is `{ "name": "Oleksandr Ieremchuk" }` with no email;
  the email lives only in `marketplace.json` `owner`, outside `plugins/`.
- No `$schema` key in `marketplace.json`.

## Consequences
Short install strings; one tag per release instead of two. The shared version
means a change to either plugin bumps both. If the plugins ever need
independent release cadences, switch to per-plugin `<name>--v<version>` tags
with a new ADR.
