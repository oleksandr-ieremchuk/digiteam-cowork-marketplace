# Decisions (ADR log)

Append-only record of material, **implemented** architecture decisions — the
"why" behind non-obvious choices. Present-tense "how it works" lives in
[../architecture.md](../architecture.md) and the references; this is history.

One file per decision: `NNNN-<slug>.md` (zero-padded, monotonic). Reversing a
decision = a new ADR + flipping the old one's status to `superseded by NNNN`.

| # | Decision | Status |
|---|----------|--------|
| 0001 | Public repository | accepted |
| 0002 | Two plugins, split by tool | accepted |
| 0003 | superpowers as a documented prerequisite, not a declared dependency | accepted |
| 0004 | Chat-only HandOff / Report protocol, no shared state | accepted |
| 0005 | Naming, versioning and author fields | accepted |
| 0006 | LF line endings enforced by `.gitattributes` | accepted |
