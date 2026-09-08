# HandOff — the chat block you hand to Claude Code

A HandOff is a **copy-paste block in chat**, one per repo. It is never written to
a file. The user opens Claude Code in the target repo and pastes it; Code does
the actual git/code work and prints a Report back (see `report-format.md`).

Keep it a pointer, not a re-derivation of the spec: Code reads the spec and the
plan itself. State the phase to run, the gate already passed, and the per-repo
acceptance criteria, then name the `superpowers` skill to use.

On the Code side the `delivery-executor` plugin (skill
`executing-delivery-handoff`) recognises the `Delivery HandOff —` header,
checks the repo's plan stage against the requested phase, runs the phase, and
prints the Report in the exact shape below. The HandOff works without it too —
the block is self-describing.

## Template

```
Delivery HandOff — <feature-slug> — repo: <repo-name>

Spec: <path to design_docs/...-design.md in this repo>
Your scope in this repo: <the slice of the feature this repo owns>
Phase to run: <0b write the plan | 2 execute | 4 deploy + functional verify | 7 review docs | 8 move done→documented>
Gate already passed: <e.g. "plan approved 2026-06-16" | "none — first handoff">

Do:
- Run this HandOff with `executing-delivery-handoff` (delivery-executor plugin) if installed.
- Use <superpowers:writing-plans | superpowers:executing-plans | the relevant step>.
- <phase-specific instruction — see below>

Acceptance criteria for this repo:
- <criterion 1>
- <criterion 2>

When done, print a Report (see below) and I'll paste it back to Cowork.

Report back:
- Phase completed and current plan stage (drafts/ongoing/done/documented)
- What changed: summary + key files
- Functional verification: what you ran and the result
- Deviations from the plan, if any
- Blockers, if any
- Ready for which gate
```

## Phase-specific instruction line

- **0b (plan):** "Read the spec, write the implementation plan for this repo's
  scope with `superpowers:writing-plans`, leave it in `drafts/`."
- **2 (execute):** "Move the plan `drafts → ongoing` and execute it with
  `superpowers:executing-plans`."
- **4 (deploy + verify):** "Deploy, run functional verification in this repo,
  then move the plan `ongoing → done`."
- **7 (review docs):** "Review the architecture-doc changes Cowork authored;
  confirm they match the real code; approve or list mismatches."
- **8 (move to documented):** "Move the plan `done → documented` and commit."

## Corrective HandOff (after a failed gate)

Same template, but `Phase to run` is the re-do, `Gate already passed` is "none —
corrective", and add a `Fix:` block listing exactly what failed the gate (the
deltas against the spec/plan, or the failing cross-repo case from integration
verification). Do not advance the feature until the new Report clears the gate.
