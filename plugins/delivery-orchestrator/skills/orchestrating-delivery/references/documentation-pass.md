# Documentation pass — phase 6

This is the documentation phase of the lifecycle. It makes **documentation a
development stage**: work isn't finished when code merges, but when the
architecture description reflects it and the decisions behind it are recorded.

You run it after the acceptance + integration gate (phase 5) has passed. You
author prose only — Claude Code reviews it (phase 7) and does the git move
`done → documented` (phase 8). You never commit here.

## Principles

- **Cowork writes the prose; Code does version control.** Author/edit docs and
  propose the file moves; the git move and commit are Code's (phases 7–8). Never
  run git here.
- **Synthesize, don't copy.** Extract each plan's *net architectural delta* (new/
  changed components, contracts, data shapes, lifecycle, cross-cutting concerns)
  and weave it into the living picture — not a changelog of the plan.
- **Capture implemented decisions.** Prose says how it works *now*; an
  append-only decision log (`architecture/decisions/`, ADR style) says *why*.
  Record a decision only once it actually **shipped** — not if it was deferred or
  superseded before landing.
- **Per-subsystem, not per-plan.** Reference docs group by subsystem (auth,
  storage, scheduling, gateway…), so the architecture reads as a coherent whole.
- **Idempotent and safe to interrupt.** A plan stays in `done/` until its delta
  is reflected and its move is handed off. Re-running with nothing new proposes
  nothing.
- **Discover, don't hardcode.** Find paths and the subsystem list by inspecting
  the repo, so this works in any repository.

## The pass — step by step

### Step 0 — Orient

Find conventions by inspection: the design/architecture home (usually
`design_docs/`, else `docs/`), with the architecture set at
`<design-home>/architecture/` and the decision log at
`architecture/decisions/`; the plan lifecycle folders
(`engineering_plans/{drafts,ongoing,done}` + the terminal `documented/`). Check
whether the convention already exists. State what you found before changing
anything.

### Step 1 — First-run scaffold (only if the convention is absent)

If `architecture/`, `architecture/decisions/`, and `documented/` don't exist,
create them: a main `architecture.md` from `references/architecture-doc-template.md`
(core sections + empty manifest), an empty `references/`, a `decisions/` with a
short `README.md` (ADR format, also in the template), and the `documented/`
plan-stage folder (with `.gitkeep`). Optionally amend the design home's
`README.md` with the lifecycle text in `references/readme-lifecycle-amendment.md`
**only if the team wants the process documented in-repo** — skip this if the
skill is being kept private to you. On a large existing backlog, the first pass
is a bootstrap: author `architecture.md` from the current state of the system,
not just one plan. If the convention already exists in the repo, skip to Step 2.

### Step 2 — Compute the undocumented backlog

The work = plans in `done/` whose slug is **not** in `documented/`. List them
(newest first or grouped by subsystem). This folder diff is the authoritative
"what still needs documenting" and survives interruptions.

### Step 3 — Extract the delta + the decisions (per plan)

For each undocumented plan, read **the plan and its paired spec**. Distil:

(a) the **net architectural change** — new/changed components and their
responsibility; public contracts (APIs, tool signatures, message/data shapes,
store layouts); changed lifecycle/control flow; cross-cutting concerns; anything
that supersedes a previously-documented fact.

(b) the **material decisions it implemented** — usually already written in the
plan's "Decisions taken" / the spec's decision tables. Keep the ones that
shipped; drop deferred/superseded-before-landing. Material = a future engineer
would otherwise ask "why this way and not the obvious other way?".

A purely internal plan (refactor, test-only) with no delta and no material
decision documents as "no architectural change" and still advances.

### Step 4 — Update the docs + record the decisions

Fold the delta in at the right altitude: basic/structural facts → `architecture.md`
core sections (keep it readable end-to-end); detail → the relevant
`references/<subsystem>.md` (new reference only for a genuinely new subsystem,
else extend); keep the manifest table in sync and correct superseded facts in
place. Record each newly-shipped material decision as an ADR at
`architecture/decisions/NNNN-<slug>.md` (next free number, append-only); a
decision that supersedes another flips the old ADR's status to `superseded by
NNNN` with a forward pointer — never edit history away. Link each ADR from the
subsystem reference it affects. Use `references/architecture-doc-template.md` for
all shapes.

Test of done: a new engineer could read `architecture.md` + the touched reference
and understand how this part works *today*, and find in `decisions/` *why* the
non-obvious choices were made — without reading the plan.

### Step 5 — Consistency check

Verify before presenting: every `references/` file appears in the manifest and
every manifest row resolves (no dangling/orphan); ADR numbering is monotonic,
every ADR has a status, every `superseded` points forward; no broken internal
links. Fix mismatches.

### Step 6 — Present the doc changes

Show the proposed doc diff (or a tight summary), the ADRs you're adding, and the
exact list of plans that will advance `done → documented`. This is the doc-review
handoff: build a HandOff (see `handoff-prompt.md`, phase 7) asking Code to
confirm the docs match the real code.

### Step 7 — Hand off the move

On Code's docs-approval (phase 7), the move `done/<slug>.* → documented/<slug>.*`
for exactly the reflected plans, plus the commit, is Code's (phase 8) — it's in
the HandOff. Do not run git yourself. Once moved, the Step 2 backlog no longer
shows those plans, so the next pass is automatically scoped to new work only.

## Properties to preserve

- **Idempotent:** re-running with no new `done/` plans changes nothing.
- **Partial-safe:** a plan is "documented" only once its move is handed off.
- **Append-only history:** ADRs are superseded with a forward pointer, never
  rewritten away. Prose is present-tense; the log is history.
