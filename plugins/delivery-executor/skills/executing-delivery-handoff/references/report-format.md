# Report — what you print back

The Report is the **only** channel from this repo to the orchestrator. Cowork
cannot run git and reads the repo read-only, so it relies on the Report for
"what changed" and on the plan-stage folder for "where the repo is". Print it as
the **last thing** in your output, inside a fenced code block, so the user can
copy it in one gesture. Never write it to a file; never commit it.

## Exact shape

```
Delivery Report — <feature-slug> — repo: <repo-name>

Phase completed: <0b | 2 | 4 | 7 | 8 | none — blocked>
Plan stage now: <drafts | ongoing | done | documented | none>
What changed: <summary> — key files: <paths>
Functional verification: <what was run> → <pass/fail + detail>
Deviations from plan: <none | list>
Blockers: <none | list>
Ready for gate: <plan review | deploy approval | acceptance | docs review | — >
```

## Filling it in

- **Phase completed** — the phase from the HandOff, or `none — blocked` if a
  precondition failed and you did nothing.
- **Plan stage now** — read it from disk after your last commit; do not report
  the stage you intended.
- **What changed** — one or two sentences plus the key files. For phase 0b this
  is the plan path; for phase 8 it is the move.
- **Functional verification** — the actual commands / checks you ran and their
  actual results. For phases without verification (0b, 8) write `n/a`. For
  phase 7 write what you compared and the outcome (approved / N mismatches).
- **Deviations from plan** — anything you did differently from the plan or the
  HandOff, with a one-line reason each. Cowork decides whether it passes.
- **Blockers** — anything that stopped you or that Cowork must resolve: a
  stage mismatch, a cross-repo contract that differs from the spec, an
  acceptance criterion you could not meet, a failing deploy.
- **Ready for gate** — from the phase map: 0b → `plan review`, 2 → `deploy
  approval`, 4 → `acceptance`, 7 → `docs review` (or `—` if you listed
  mismatches), 8 → `—`. If blocked, `—`.

Optionally, before the Report, list the acceptance criteria with met / not met
and the evidence for each — Cowork uses that at the gate.

## Corrective runs

Same shape. `Phase completed` is the phase you re-did; add the fixed items to
`What changed`; `Ready for gate` is the gate that previously failed.
