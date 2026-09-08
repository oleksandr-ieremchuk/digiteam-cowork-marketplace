# README Lifecycle Amendment

Add (or adapt) this to the design home's `README.md` during first-run scaffold,
so the documentation stage is discoverable in-repo. Use **only** if the team
wants the process documented in the repo; skip it while the skill is private to
you. Match the repo's existing wording; this is the intent, not literal copy.

```markdown
## Documentation is a development stage

Work is finished when the architecture reflects it and the decisions behind it
are recorded — not when the code merges.

Lifecycle:

  brainstorm
    -> design_docs/<topic>-design.md            (spec)
    -> engineering_plans/drafts -> ongoing -> done   (implementation)
    -> (at work-acceptance) architecture/ + decisions/ updated + plan -> engineering_plans/documented/

- `architecture/` — the living, skill-shaped description of how the system is
  built today: a concise `architecture.md` (overview + core building blocks + a
  manifest of references) plus per-subsystem docs under `architecture/references/`.
  Present state, not history.
- `architecture/decisions/` — the append-only ADR log: why the non-obvious,
  **implemented** choices were made. Reversing a choice adds a new ADR and marks
  the old one superseded; nothing is rewritten away.
- `engineering_plans/done/` — shipped. `engineering_plans/documented/` — shipped
  **and** folded into `architecture/` + `decisions/`.

### The acceptance pass

When completed work is handed back, run the documentation pass: read each plan in
`done/` not yet in `documented/` (plus its paired spec), fold the net
architectural delta into `architecture.md` and/or the relevant
`architecture/references/<subsystem>.md`, record the decisions it implemented as
ADRs under `architecture/decisions/`, update the manifest, then move the plan
`done/ -> documented/`. The pass is per-subsystem, idempotent, and safe to run in
batches. (Prose is authored in Cowork; the git move is done by whoever owns
version control.)
```
