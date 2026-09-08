---
name: executing-delivery-handoff
description: >
  Use in Claude Code when the user pastes a "Delivery HandOff — <slug> — repo:
  <name>" block from Cowork, or says "run this handoff", "here's the handoff",
  "execute the delivery handoff", "print the delivery report", "move the plan to
  ongoing/done/documented", and equivalents in the user's language. You are the
  single-repo executor in a multi-repo delivery process: you verify the repo and
  plan stage match the requested phase, run exactly that phase (write the plan,
  execute it, deploy + verify, review docs, or move the plan to documented),
  keep the plan-stage folders truthful with git, and finish by printing a
  Delivery Report in the exact format Cowork parses. Never widen scope beyond
  the HandOff.
---

# Executing a delivery HandOff

Cowork orchestrates a feature across repositories and decides the gates. You are
the executor **inside one repository**: everything that touches code and git
here is yours — the plan, the implementation, deploy and functional
verification, and every move of the plan between stage folders. You do one
phase per HandOff, then print a Report the user pastes back to Cowork.

The two sides talk only through chat. A HandOff is text you receive; a Report
is text you print. Neither is ever written to a file.

## Step 1 — Parse the HandOff

Extract from the pasted block (shape in `references/handoff-format.md`):

- **slug** and **repo** from the header line;
- **spec** path, **scope** in this repo, **phase to run** (0b / 2 / 4 / 7 / 8),
  **gate already passed**;
- **acceptance criteria** (the checklist you must satisfy);
- an optional **Fix:** block — present only on a corrective HandOff.

If a field you need is missing, ask before doing anything.

## Step 2 — Verify preconditions (stop if any fails)

1. **Right repo.** The current working directory is the repo named in the
   header (check the folder name and `git remote -v`). If not, stop and say so —
   never run another repo's HandOff here.
2. **Spec present** at the given path. Read it; find the slug and this repo's
   scope in it.
3. **Plan stage matches the phase.** Discover the plan lifecycle folders by
   inspection (usually `engineering_plans/{drafts,ongoing,done,documented}`;
   never hardcode) and locate the plan carrying the slug. The required starting
   stage per phase is in `references/phase-map.md`. A mismatch (for example
   phase 2 requested but the plan is already in `ongoing/`, or phase 4 requested
   but there is no plan) means Cowork's picture of this repo is stale: **do not
   guess** — print a Report with the mismatch under `Blockers` and stop.
4. **Clean tree.** `git status` shows no unrelated uncommitted changes. If it
   does, ask the user how to proceed rather than mixing work.

State what you found (repo, spec, plan stage, phase) before acting.

## Step 3 — Run exactly the requested phase

Details per phase in `references/phase-map.md`. In short:

- **0b — write the plan.** Read the spec; write the implementation plan for
  *this repo's scope only* with `superpowers:writing-plans`; save it in
  `drafts/` named by the slug; commit.
- **2 — execute.** `git mv` the plan `drafts → ongoing` and commit that move
  first. Then execute the plan with `superpowers:executing-plans`, committing as
  the plan prescribes. Do not deploy and do not move the plan to `done/` — that
  is phase 4, after Cowork approves the changes.
- **4 — deploy + functional verification.** Deploy per this repo's own
  convention; run functional verification inside this repo (tests, smoke
  checks, whatever the plan and acceptance criteria call for); then `git mv`
  the plan `ongoing → done` and commit. If verification fails, do not move the
  plan; report the failure.
- **7 — review docs.** Cowork has authored architecture prose / ADRs. Read
  them against the real code and list every mismatch; approve only when there
  are none. No plan move in this phase.
- **8 — move to documented.** `git mv` the plan `done → documented` and commit.
  Nothing else.

**Corrective HandOff** (`Gate already passed: none — corrective`, with a `Fix:`
block): address the `Fix:` items only. Do not redo the phase from scratch and
do not move the plan; the stage stays where it is until the gate passes.

## Step 4 — Check the acceptance criteria

Go through the HandOff's acceptance criteria one by one and record met / not
met with evidence (a command you ran, a file you changed, a test that passed).
An unmet criterion is either a deviation you can justify or a blocker — never
silently dropped.

## Step 5 — Print the Report (always, last)

End every run — success, partial, or blocked — by printing the Report per
`references/report-format.md`, in a fenced code block so the user can copy it
in one gesture. It must be the last thing in your output. Do not write it to a
file and do not commit it.

## Hard rules

- **One repo, one phase, one HandOff.** Never touch another repository; never
  run the next phase "while you're at it"; never advance the plan stage beyond
  what the phase specifies.
- **The stage folder is the truth.** Every stage move is a `git mv` plus a
  commit, done at the moment the phase says. Cowork gates on that folder.
- **Stay in scope.** Implement what the spec assigns to this repo and what the
  plan says. Cross-repo contracts you depend on (an API another repo ships, a
  shared version) are taken as given from the spec — if reality differs, that
  is a blocker for Cowork's integration gate, not something to patch around.
- **Report deviations honestly.** If you departed from the plan, say what and
  why. Cowork decides whether it passes the gate.
- **Never fabricate verification.** "Functional verification" in the Report
  lists commands actually run and their actual results.

## References

- `references/handoff-format.md` — the HandOff block you receive, field by field.
- `references/phase-map.md` — required starting stage, actions, and end stage per phase.
- `references/report-format.md` — the exact Report to print.
