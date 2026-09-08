# Phase map — owner, gate, tool

The canonical lifecycle. Phase 0 is the spec/plan setup; phases 1–8 are the eight
delivery steps. "Owner" is who does the work; "gate" is the explicit approval that
must pass before the feature advances. Cowork-owned gates are decided by you;
Code-owned phases happen in Claude Code and you only see them via a Report.

| # | Phase | Owner | Tool / artifact | Gate |
|---|-------|-------|-----------------|------|
| 0a | Brainstorm + spec (across all target repos) | Cowork | `superpowers:brainstorming` → spec in `design_docs/` (lists target repos + per-repo scope) | — |
| 0b | Plan from spec | Code (per repo) | HandOff → Code runs `superpowers:writing-plans` → plan in `drafts/` | — |
| 1 | Plan review | Cowork | read each draft plan; check it against the spec | **plan approved** |
| 2 | Execution | Code (per repo) | plan `drafts → ongoing`; `superpowers:executing-plans` | — |
| 3 | Change review + deploy approval | Cowork | read Code's Report + resulting files (read-only) against the plan | **deploy approved** |
| 4 | Deploy + functional verification | Code | deploy, verify in-repo, plan `ongoing → done`, print a Report | — |
| 5 | Acceptance + integration verification | Cowork | cross-repo, read-only, end-to-end (the join point) | **acceptance approved** |
| 6 | Documentation | Cowork | documentation pass (`references/documentation-pass.md`), prose only, no commit | — |
| 7 | Documentation review | Code | confirm the docs match the real code | **docs approved** |
| 8 | Plan `done → documented` | Code | git move + commit | — |

## Notes

- **Gate failure → corrective HandOff, no advance.** If a Cowork-owned gate
  (1, 3, 5) fails, describe the delta in a HandOff to the relevant repo, wait for
  a new Report, and re-check the same gate. Looping here is normal.
- **Multi-repo.** Phases 0b–4 and 7–8 run per repo, in parallel. Phase 5 is the
  cross-repo join — it starts only when **every** target repo has reported `done`.
  The feature is finished only when **every** target repo reaches `documented`.
- **The plan-stage folder is the durable phase signal** for a repo: `drafts`
  (planned), `ongoing` (executing), `done` (shipped + functionally verified),
  `documented` (folded into architecture). Cowork reads the folder; only Code
  moves a plan between stages.
