# HandOff — the block you receive

Cowork pastes one of these per repo. It is chat text, never a file. Treat it as
the authoritative statement of *what to do in this repo now*; treat the spec and
the plan it points to as the authoritative statement of *what this repo must deliver*.

It normally reaches you as the argument of your own slash command — the block's
first line is `/delivery-executor:executing-delivery-handoff`, so the rest
arrives as `$ARGUMENTS`. It may also arrive as a plain pasted message, or the
user may run the slash command with nothing after it; in that last case ask
them to paste the HandOff before doing anything.

## Shape

```
/delivery-executor:executing-delivery-handoff
Delivery HandOff — <feature-slug> — repo: <repo-name>

Spec: <design_docs/<feature-slug>-design.md — this repo's own spec>
Phase to run: <0b write the plan | 2 execute | 4 deploy + functional verify | 7 review docs | 8 move done→documented>
Gate already passed: <e.g. "plan approved 2026-06-16" | "none — first handoff" | "none — corrective">

Do:
- Use <superpowers:writing-plans | superpowers:executing-plans | the relevant step>.
- <phase-specific instruction>

Fix:                      ← only on a corrective HandOff
- <exactly what failed the gate>

Acceptance criteria for this repo:
- <criterion 1>
- <criterion 2>

When done, print a Report and I'll paste it back to Cowork.

Report back:
- <the fields Cowork expects — see report-format.md>
```

## Field by field

| Field | What you do with it |
|---|---|
| slash line | Your own invocation; carries no data. Ignore it when parsing. |
| header `<slug>` / `<repo>` | Identify the feature and confirm you are in the right repo. Every artifact you create carries the slug. |
| `Spec` | This repo's own spec; everything in it is yours. Read it first. It never needs another repo — a path or link into one is a blocker. |
| `Phase to run` | The one phase you execute. Required starting plan stage and end stage are in `phase-map.md`. |
| `Gate already passed` | Tells you the gate Cowork cleared to send this. `none — corrective` means a gate **failed** and this is the re-do. |
| `Do:` | Which `superpowers` skill and phase-specific instruction to follow. |
| `Fix:` | Corrective only: the exact delta that failed the gate. Address these items and nothing more. |
| `Acceptance criteria` | Your checklist. Each one is met-with-evidence, a justified deviation, or a blocker in the Report. |
| `Report back:` | The fields Cowork will parse. Print them in the Report shape from `report-format.md`. |

## When the HandOff and disk disagree

The plan-stage folder in the repo is the durable truth. If the HandOff asks for
a phase whose starting stage does not match what is on disk, Cowork's picture
is stale. Do not "fix" it by moving the plan — print a Report with the mismatch
under `Blockers` and stop. Cowork will recompute and send a new HandOff.
