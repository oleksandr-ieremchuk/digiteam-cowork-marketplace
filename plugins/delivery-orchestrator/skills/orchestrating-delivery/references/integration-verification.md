# Integration verification — the cross-repo gate (phase 5)

This is the gate only Cowork can run, because only Cowork sees all the feature's
repos at once. Claude Code already did **functional** verification inside each
repo (phase 4); this is **integration** verification: do the pieces shipped in
different repos actually fit together?

It is a **method, not a fixed script** — the contracts depend on the feature.

## Preconditions

- Every target repo has reported `Plan stage now: done` (and the plan files
  confirm it). If any repo is behind, the gate is **blocked** — say which repo
  you're waiting on and stop.

## Method

1. **Derive the cross-repo contracts from the spec.** List every place where one
   repo depends on something another repo shipped: tool/function signatures,
   API or message shapes, shared version numbers or guide versions, file/path
   conventions, config or env contracts.
2. **Exercise each contract end-to-end across the mounted repos**, read-only:
   - load or build the artifacts and check the consumer side resolves the
     producer side (e.g. repo A references a tool/version that repo B now ships);
   - run non-mutating smoke checks (the plugin loads, a skill's trigger phrases
     resolve, a referenced path exists, a schema validates);
   - trace the data/handoff path that crosses the repo boundary and confirm both
     ends agree.
3. **Record pass/fail per contract**, with the evidence you saw.

## Hard constraints

- **Read-only and non-mutating only.** No writes into the repos, no git, no
  deploys, no code edits. If a check would change state, it belongs in a HandOff
  to Code, not here.
- **No git.** Inspect files directly; never run `git`/`gh`. A sandbox git error
  is a Cowork bug — ignore it.

## Outcome

- **All contracts pass →** acceptance approved. Proceed to phase 6
  (the documentation pass, `documentation-pass.md`).
- **Any contract fails →** do not advance. Issue a corrective HandOff to the repo
  on the failing side (producer or consumer), describing the exact mismatch as
  the `Fix:` block. Re-run this gate when its new Report comes back.
