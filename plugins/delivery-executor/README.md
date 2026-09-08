# delivery-executor

**Install this into Claude Code.** It is the executing half of the DigiTeam
delivery process: it runs one phase, in one repository, from one HandOff pasted
out of Cowork, where the companion plugin `delivery-orchestrator` produced it.

It contains one skill, **`executing-delivery-handoff`**, which triggers on a
pasted `Delivery HandOff — <slug> — repo: <name>` block, and on phrases like
"run this handoff", "here's the handoff", "execute the delivery handoff",
"print the delivery report", "move the plan to ongoing/done/documented" — and
their equivalents in other languages.

It verifies the repo and the plan's stage folder against the requested phase,
runs exactly phase 0b (write the plan), 2 (execute), 4 (deploy + functional
verification), 7 (review docs) or 8 (move `done → documented`), keeps the stage
folders truthful with `git mv` plus a commit, and finishes by printing a
Delivery Report for the user to paste back into Cowork. It never widens scope
beyond the HandOff.

Requires the `superpowers` marketplace (`superpowers:writing-plans`,
`superpowers:executing-plans`).

Install instructions, the full phase table and the repo conventions are in the
[marketplace README](../../README.md).
