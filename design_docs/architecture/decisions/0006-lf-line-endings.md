# 0006 — LF line endings enforced by `.gitattributes`

- **Status:** accepted
- **Date:** 2026-09-08
- **Source:** plan `marketplace-bootstrap` (Global Constraints, Task 3); spec §3.1 as amended at gate 1

## Context
The skill files are shipped bytes: the plan verified them by sha256 and the
integration gate diffs the two Report blocks byte-for-byte. The first
maintainer's machine runs Git for Windows with `core.autocrlf=true` in system
config, which would rewrite every text file to CRLF on checkout and back on
commit — breaking byte-identity claims and producing spurious diffs. A repo-local
`core.autocrlf=false` fixes one clone but does not travel with a public repo.

## Decision
A tracked `.gitattributes` with `* text=auto eol=lf`, in addition to the local
config on the maintainer's clone.

## Consequences
Every clone, on any OS, checks out LF and commits LF; sha256 checks and the
parity diff stay meaningful. Contributors' editors must tolerate LF on Windows,
which modern ones do.
