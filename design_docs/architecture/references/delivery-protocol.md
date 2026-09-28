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

**HandOff** opens with the executor's own slash command,
`/delivery-executor:executing-delivery-handoff`, alone on the first line. The
header `Delivery HandOff — <slug> — repo: <repo>` follows, then `Spec`,
`Phase to run` (one of 0b / 2 / 4 / 7 / 8), `Gate already passed`, a `Do:` list naming the superpowers skill to use, an optional `Fix:`
block (corrective HandOffs only), `Acceptance criteria for this repo`, and a
`Report back:` list of the fields expected. The slash line carries no data; the
executor ignores it when parsing. `Spec` is always this repo's own
`design_docs/<slug>-design.md`, and that spec is the whole scope, so there is
no separate scope field and a HandOff never names another repo's files. The
phase-7 HandOff lists every doc file Cowork authored, each with its sha256;
the phase-8 HandOff repeats that list for Code to commit.

**How the HandOff arrives.** Pasted as one message, the first line invokes the
executor skill and the rest of the block reaches it as that command's argument.
Two fallbacks are part of the contract rather than accidents: the block pasted
without its first line is still self-describing, and the skill stays
model-invocable so it can pick it up; and the command sent with nothing after it
makes the skill ask for the block and stop, instead of guessing.

**Report** header `Delivery Report — <slug> — repo: <repo>`, then exactly these
fields in this order: `Phase completed` (0b / 2 / 4 / 7 / 8 / `none —
blocked`), `Plan stage now` (drafts / ongoing / done / documented / `none`),
`What changed`, `Functional verification`, `Deviations from plan`, `Blockers`,
`Ready for gate` (plan review / deploy approval / acceptance / docs review / —).

**Spec convention.** Every target repo holds its own spec,
`design_docs/<slug>-design.md`, describing only the changes to make there, with
its Provides / Consumes contracts written out. A multi-repo feature also has a
feature record, `design_docs/<slug>-feature.md`, in one primary repo: the only
file allowed to talk about several repos, holding the target-repo list and the
cross-repo contract matrix. A single-repo feature has no feature record; its
design file doubles as one.

**Shared vocabulary.** Phases 0a–8 with owners as in the phase map; the gate
**specs consistent** closes 0a (contract matrix against each repo's Provides /
Consumes, and every spec self-contained); Cowork-owned gates 1, 3, 5 (and 7 for
Code); plan stages `drafts → ongoing → done →
documented`, moved only by Code, each move its own commit. The slug ties every
artifact of a feature together across repos.

**Parity rule.** Both HandOff blocks open with the slash command line. The
Report block in the two `report-format.md` files must be byte-identical between
the header line and `Ready for gate`; the HandOff template and the HandOff shape
must carry the same fields in the same order — with one allowed difference: the
optional `Fix:` block (corrective HandOffs only) sits inline in the executor's
shape and in a separate "Corrective HandOff" section of the orchestrator's
template. Annotations may differ. This is the cross-plugin check the
orchestrator's integration-verification gate runs on this repository.

## Lifecycle / flow

1. Cowork finds the target repos (from the feature record, or the single
   repo holding the design file; it asks when that is ambiguous), computes the
   feature's phase from each repo's stage folders plus the latest pasted Report,
   and emits a HandOff for the phase Code owns next.
2. The user pastes it into Claude Code in the target repo as a single message.
   Its first line invokes the executor skill and hands it the rest. The skill
   verifies repo, spec and plan stage against the requested phase; on mismatch
   it prints a Report with `Phase completed: none — blocked` and stops.
3. Code runs exactly that phase, moves the plan stage where the phase says,
   and prints the Report as the last thing in its output. At phase 7 the listed
   doc files are uncommitted by design; any other uncommitted change in the
   design home blocks. At phase 8 Code commits exactly those files, then moves
   the plan `done → documented` in its own commit.
4. The user pastes the Report back. Cowork decides the gate: pass → next
   HandOff; fail → a corrective HandOff with a `Fix:` block, same gate re-run.

## Constraints & decisions

- Chat-only, no shared files — [ADR 0004](../decisions/0004-chat-only-handoff-report-protocol.md).
- Two plugins rather than one, so each tool holds only its half of the rules —
  [ADR 0002](../decisions/0002-two-plugins-split-by-tool.md).
- Invocation is deterministic by slash command, not by description matching —
  [ADR 0007](../decisions/0007-slash-command-first-line-of-every-handoff.md).
- Specs and docs are repo-local; the spec is the scope —
  [ADR 0008](../decisions/0008-repo-local-specs-and-docs.md).
- The gate "specs consistent" closes 0a —
  [ADR 0009](../decisions/0009-specs-consistent-gate.md).
- Docs are authored in the working tree, reviewed by hash, committed at phase 8 —
  [ADR 0010](../decisions/0010-working-tree-docs-and-phase-8-commit.md).
- The redundant copies are a maintenance cost accepted on purpose; the parity
  rule and the integration gate are what keep them honest.
