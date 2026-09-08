# Delivery protocol

## Responsibility

The protocol is the whole interface between the two plugins. It defines what
Cowork hands to Claude Code (a **HandOff**), what Claude Code hands back (a
**Report**), and the shared vocabulary both sides use to name phases, gates and
plan stages. It deliberately does *not* define any shared storage: there is no
file, database or API between the tools — the user copies text from one chat
into the other.

## Structure

```
Cowork (delivery-orchestrator)                 Claude Code (delivery-executor)
  spec, gates, integration verify                one phase in one repo
        │                                              ▲
        │  HandOff — chat block, one per repo ─────────┘
        ▲                                              │
        └───────── Report — chat block, printed last ──┘
```

Each side carries its own copy of both block shapes:

| Block | Orchestrator copy (author) | Executor copy (consumer) |
|---|---|---|
| HandOff | `orchestrating-delivery/references/handoff-prompt.md` — the template Cowork fills | `executing-delivery-handoff/references/handoff-format.md` — the same block, field by field, from the reader's side |
| Report | `orchestrating-delivery/references/report-format.md` — how Cowork parses it | `executing-delivery-handoff/references/report-format.md` — how Code fills it |

## Contracts

**HandOff** header `Delivery HandOff — <slug> — repo: <repo>`, then `Spec`,
`Your scope in this repo`, `Phase to run` (one of 0b / 2 / 4 / 7 / 8), `Gate
already passed`, a `Do:` list naming the superpowers skill to use, an optional
`Fix:` block (corrective HandOffs only), `Acceptance criteria for this repo`,
and a `Report back:` list of the fields expected.

**Report** header `Delivery Report — <slug> — repo: <repo>`, then exactly these
fields in this order: `Phase completed` (0b / 2 / 4 / 7 / 8 / `none —
blocked`), `Plan stage now` (drafts / ongoing / done / documented / `none`),
`What changed`, `Functional verification`, `Deviations from plan`, `Blockers`,
`Ready for gate` (plan review / deploy approval / acceptance / docs review / —).

**Shared vocabulary.** Phases 0a–8 with owners as in the phase map; Cowork-owned
gates 1, 3, 5 (and 7 for Code); plan stages `drafts → ongoing → done →
documented`, moved only by Code, each move its own commit. The slug ties every
artifact of a feature together across repos.

**Parity rule.** The Report block in the two `report-format.md` files must be
byte-identical between the header line and `Ready for gate`; the HandOff
template and the HandOff shape must carry the same fields in the same order —
with one allowed difference: the optional `Fix:` block (corrective HandOffs
only) sits inline in the executor's shape and in a separate "Corrective
HandOff" section of the orchestrator's template. Annotations may differ. This is the cross-plugin check the orchestrator's
integration-verification gate runs on this repository.

## Lifecycle / flow

1. Cowork computes the feature's phase from the target repo's stage folders
   plus the latest pasted Report, and emits a HandOff for the phase Code owns
   next.
2. The user pastes it into Claude Code in the target repo. The executor skill
   verifies repo, spec and plan stage against the requested phase; on mismatch
   it prints a Report with `Phase completed: none — blocked` and stops.
3. Code runs exactly that phase, moves the plan stage where the phase says,
   and prints the Report as the last thing in its output.
4. The user pastes the Report back. Cowork decides the gate: pass → next
   HandOff; fail → a corrective HandOff with a `Fix:` block, same gate re-run.

## Constraints & decisions

- Chat-only, no shared files — [ADR 0004](../decisions/0004-chat-only-handoff-report-protocol.md).
- Two plugins rather than one, so each tool holds only its half of the rules —
  [ADR 0002](../decisions/0002-two-plugins-split-by-tool.md).
- The redundant copies are a maintenance cost accepted on purpose; the parity
  rule and the integration gate are what keep them honest.
