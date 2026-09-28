# State discovery — computing the current phase

Cowork holds no external process state. Each run, recompute where the feature is
from what's on disk plus the latest pasted Report. This is what makes the skill
idempotent and safe to resume after any interruption.

## Inputs (all read-only)

- The **feature slug** — declared in the spec, else the spec's filename slug.
  Every artifact for the feature carries it.
- The **target repos**, found across the mounted repos:
  - `<slug>-feature.md` in exactly one repo → that repo is the primary; read the
    target-repo list from it.
  - No feature file, and `<slug>-design.md` in exactly one repo → a single-repo
    feature; that repo is the primary and its design file doubles as the feature
    record.
  - Zero or several feature files (or no feature file and several design files)
    → ask the user; never guess.
- For each **target repo**:
  - Is this repo's own spec, `<slug>-design.md`, present in `design_docs/` (or
    the repo's design home)?
  - Is there a plan for the slug, and in which stage folder:
    `drafts` / `ongoing` / `done` / `documented`?
- The most recent **Report** the user pasted for each repo (chat, transient).

## Per-repo phase from the plan stage

| What you find | Repo phase |
|---|---|
| this repo's `<slug>-design.md` exists, no plan | 0b — awaiting plan (send/await HandOff), once **specs consistent** has passed |
| plan in `drafts/` | 1 — plan review (Cowork gate) |
| plan in `ongoing/` | 2 — executing (Code) |
| plan in `done/` | 3→5 — change review / deploy / acceptance, per latest Report |
| plan in `documented/` | done for this repo |

The pasted Report refines a `done`-stage repo: its `Ready for gate` tells you
whether you're at change review (phase 3), or the work is deploy-verified and
waiting for the cross-repo acceptance gate (phase 5).

## Between 0a and 0b — the gate "specs consistent"

Specs on disk but no plan anywhere, and the gate **specs consistent** not yet
passed → the feature is still in 0a. Run that gate: every contract-matrix row
appears, with the identical shape, in the producer's Provides and the
consumer's Consumes, and every per-repo spec is self-contained. Single-repo:
self-containment only. Send no 0b HandOff until it passes.

## Feature phase = the aggregate

- The feature cannot pass the **phase-5 integration gate** until **every** target
  repo is at `done`. If repos are uneven, report each repo's phase and name the
  one you're waiting on.
- The feature is **finished** only when **every** target repo reaches
  `documented`.

## Discipline

- Discover paths by looking at the repo (design home, plan-stage folders); don't
  hardcode — same rule as the documentation pass (`documentation-pass.md`).
- If disk and a Report disagree, disk wins for gating (the stage move is the
  durable fact); note the discrepancy.
- Always **state what you found** — per-repo stage and the derived phase — before
  acting or issuing a HandOff.
