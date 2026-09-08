# Phase map — the executor's view

The full lifecycle has nine phases (0a–8). Cowork owns the spec (0a), the review
gates (1, 3, 5) and the documentation prose (6). You own the phases that touch
code and git inside this repo: **0b, 2, 4, 7, 8**. You will only ever be handed
one of those five.

| Phase | Plan stage required at start | What you do | Plan stage at end | Cowork's next gate |
|---|---|---|---|---|
| **0b** write the plan | spec present, **no plan** for the slug | `superpowers:writing-plans` from the spec, scoped to this repo; save in `drafts/`; commit | `drafts` | 1 — plan review |
| **2** execute | `drafts` | `git mv drafts → ongoing`, commit; `superpowers:executing-plans`; commit as the plan prescribes; **no deploy, no move to done** | `ongoing` | 3 — change review + deploy approval |
| **4** deploy + functional verify | `ongoing` | deploy per repo convention; run in-repo functional verification; `git mv ongoing → done`, commit | `done` | 5 — acceptance + integration verification (cross-repo) |
| **7** review docs | `done` | read Cowork's architecture prose / ADRs against the real code; approve or list mismatches; **no move** | `done` | — (Cowork fixes docs or sends phase 8) |
| **8** move to documented | `done` (docs approved) | `git mv done → documented`, commit | `documented` | — feature done for this repo |

## Plan lifecycle folders

Discover them by inspection; do not hardcode. The common convention is
`engineering_plans/{drafts,ongoing,done,documented}/` next to `design_docs/`. If
the repo has the convention, use it exactly. If it does **not** exist yet and you
are running phase 0b, create the four folders (with `.gitkeep`) as part of that
phase and mention it under `What changed`. The plan file is named by the slug
(e.g. `engineering_plans/drafts/<slug>-plan.md`) so Cowork can find it by name.

## Stage moves are commits

Each move is a `git mv` followed by its own commit (message like
`plan(<slug>): drafts → ongoing`). Cowork gates on the folder, not on your
words — if the move isn't committed, the phase isn't done.

## Corrective HandOffs

`Gate already passed: none — corrective` + a `Fix:` block. The plan stays in its
current folder. You address the listed deltas, commit, re-run the verification
that is relevant, and print a Report saying which gate you are ready for again.
No stage move unless the phase you are re-doing ends in one *and* the fix
completes it (for example, a corrective phase 4 that now passes verification
does move `ongoing → done`).

## What is never yours

- Deciding a gate. You report; Cowork approves.
- Touching another repository, or working around a cross-repo contract that
  turned out different from the spec — that is a blocker for Cowork.
- Writing HandOffs or Reports to files.
