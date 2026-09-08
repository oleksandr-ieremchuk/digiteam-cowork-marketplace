# Report — what Code prints back, and how you read it

A Report is **text Claude Code prints** at the end of its run; the user pastes it
back into Cowork. It is never a file. It is how a single-repo Code session tells
you (the cross-repo orchestrator) what happened, so you can decide the next gate
without touching git yourself.

## Expected shape

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

## How to use it

- **`Phase completed: none — blocked`** means Code did nothing: a precondition
  failed (wrong repo, spec missing, plan stage on disk did not match the phase
  you asked for, dirty tree). Re-run state discovery, fix your picture of the
  repo, and send a fresh HandOff — do not treat it as a failed gate.

- **Trust the Report for what you can't see safely.** It is your read on "what
  changed" without running git. Cross-check the claims you can against the files
  (read-only) and the plan.
- **Drive the gate from it.** "Ready for gate: deploy approval" means you run the
  phase-3 review (Report + resulting files vs the plan) and then approve or send
  a corrective HandOff.
- **Reconcile with disk.** The plan-stage folder is the durable signal; the
  Report is the fresher detail that may precede a file move. If they disagree
  (e.g. Report says `done` but the plan is still in `ongoing`), the move hasn't
  happened yet — treat the work as not-yet-`done` for gating, and note it.
- **Blockers / deviations** feed the loop: if they break a gate, answer with a
  corrective HandOff rather than advancing.
- **Multi-repo:** record each repo's Report against the feature slug. The phase-5
  integration gate waits until every target repo's Report reads `Plan stage now:
  done` (and the plan files confirm it).

If a pasted Report is missing fields you need for a gate, ask the user to get
Code to print them — don't guess.
