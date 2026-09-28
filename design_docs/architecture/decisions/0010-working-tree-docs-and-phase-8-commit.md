# 0010 — Docs authored in the working tree, reviewed by hash, committed at phase 8

- **Status:** accepted
- **Date:** 2026-09-28
- **Source:** `design_docs/repo-local-docs-design.md` (D10, D11); plan
  `repo-local-docs` and its gate-3 corrective commit, released as `v0.1.2`

## Context
While finishing `executor-trigger`, Cowork authored phase-6 docs from the remote
copy of the repo while an earlier uncommitted draft already sat in the working
tree, producing two ADR 0007 files under different names. The executor's
clean-tree check caught it, but that check also fired on the listed docs
themselves, which are uncommitted by design. And no phase said who commits the
approved docs: phase 8 was "move the plan, nothing else".

## Decision
- Cowork authors phase-6 docs in the repo's working tree, reads the design home
  there first, continues an existing uncommitted draft instead of writing a
  parallel one, and counts the next ADR number on disk, uncommitted files
  included.
- The phase-7 HandOff lists every authored file with its sha256. The executor
  checks the hashes, treats those files as expected-uncommitted, and blocks on
  any other uncommitted change in the design home. It then reviews the docs
  against the code, for self-containment, and against the real interfaces in
  Contracts & integrations.
- Phase 8 commits exactly the approved files, hash-checked, and then moves the
  plan `done → documented` in its own commit.

## Consequences
One authoritative draft per feature, a byte-exact trail from review to commit,
and no ad-hoc instructions needed to get docs into history. The executor's
phase-7 precondition now depends on the HandOff's file list being complete.
