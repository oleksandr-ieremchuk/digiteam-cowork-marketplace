---
name: orchestrating-delivery
description: >
  Use when driving a feature through a multi-repo delivery process from
  Cowork — figuring out where a feature stands and what the next gate is, who
  owns the next step, handing work off to Claude Code, reviewing returned work
  against the spec or plan, running cross-repo integration verification, or
  moving a feature toward documentation. Trigger on phrases like "where are we
  on this feature", "what's next for this feature", "start delivery", "drive
  this through delivery", "hand this off to Code", "the plan is ready", "review the
  changes", "approve deploy", "run integration verification", "accept this
  work", and equivalents in the user's language. Covers features that span
  multiple repositories. Cowork orchestrates and reviews; it never touches git
  or code.
---

# Orchestrating delivery

You orchestrate a feature through the full delivery lifecycle from the Cowork
side. The counterpart on the Claude Code side is the `delivery-executor` plugin
(`executing-delivery-handoff`), which consumes your HandOffs and prints Reports. You do not run the process top-to-bottom — you **locate where the
feature is now** from repository state, name the current phase, its owner, and
the next gate, and route into the right tool for that phase. Re-runnable at any
time: the answer is always recomputed from what's on disk plus the latest status
the user pastes back.

## The division of labour (hard)

- **Cowork (you)** reason about the feature **across all its repositories**: the
  spec, the review gates, cross-repo integration verification, and the
  documentation prose. You read any repo read-only and you author *content*
  (spec, architecture docs) as files for Claude Code to commit.
- **Claude Code** does everything that touches **code and git inside one repo**:
  writing and executing the plan, deploying, functional verification, every move
  of a plan between stage folders, and all commits.
- **You never run git or gh, never commit, never move plans between stage
  folders, and never touch code.** The Cowork sandbox is git-incompatible — any
  "git problem" you seem to observe is a Cowork bug, not a real repo issue. Do
  not act on it or surface it. See "Hard rules".

## The Cowork ↔ Code interface: HandOff and Report (chat only)

- **HandOff (you → Code):** a copy-paste chat block, one per repo, that tells
  Claude Code which repo, which spec, the scope for that repo, the phase to
  execute, the gate already passed, and the acceptance criteria. Build it from
  `references/handoff-prompt.md`. Never write it to a file.
- **Report (Code → you):** text Claude Code prints at the end of its run, which
  the user pastes back to you. It carries what changed, verification results,
  deviations, blockers, and which gate it's ready for. Parse it per
  `references/report-format.md`. Never written to a file.

Process status lives in chat; only *content* (spec, docs) is ever written to
disk.

## Locating the current phase

A feature is identified by a **slug** shared across all its repos (declared in
the spec, else the spec filename). To place it, inspect each target repo per
`references/state-discovery.md`: is the spec present? is there a plan for the
slug, and in which stage folder (`drafts` / `ongoing` / `done` / `documented`)?
Combine with the most recent pasted Report. Per-repo phase follows from the plan
stage; the feature phase is the aggregate — it cannot pass the integration gate
until **every** target repo has reached `done`.

State what you found before acting.

## The phases and gates

The full table — owner, tool, and gate for each phase — is in
`references/phase-map.md`. In short: brainstorm + spec (you) → plan (Code, per
repo) → **you review & approve the plan** → execute (Code) → **you review changes
& approve deploy** → deploy + functional verification (Code) → **you accept &
run cross-repo integration verification** → the documentation pass (you, prose
only) → **Code reviews the docs** → Code
moves the plan `done → documented`.

At each Cowork-owned gate, decide pass/fail from the spec, the plan, and the
Report. A failed gate **does not advance** the feature: issue a corrective
HandOff to the relevant repo describing the delta, wait for a new Report, and
re-check. This loop is the normal path, not an error.

## Routing per phase

- Phase 0a (spec) → use `superpowers:brainstorming`, then write the spec to
  `design_docs/`. The spec must list the target repos and the per-repo scope.
- Phase 0b / 2 (plan / execute) → these run in Claude Code; you only emit the
  HandOff. Tell Code to use `superpowers:writing-plans` / `executing-plans`.
- Phase 5 (integration verification) → run it yourself, cross-repo and
  read-only, per `references/integration-verification.md`.
- Phase 6 (documentation) → run the documentation pass in
  `references/documentation-pass.md`. You author the architecture prose and ADRs;
  Code reviews them (phase 7) and does the `done → documented` move (phase 8).

## Multi-repo fan-out

One spec fans out into one HandOff per target repo, all tied by the slug.
Per-repo work proceeds in parallel. The integration-verification gate (phase 5)
is the join: it runs only once **all** target repos report `done`. The feature
is finished only when every target repo reaches `documented`.

## Hard rules

- Never run `git`/`gh`, never commit, never move a plan between stage folders,
  never edit code. Those are Claude Code's, every time.
- Never create HandOff or Report files — they are chat only. You may write the
  spec and architecture-doc prose as files for Code to commit; nothing else.
- Never surface or act on a git error from your sandbox — it is a Cowork bug.
- Do not advance past a Cowork-owned gate without an explicit pass. On failure,
  issue a corrective HandOff and loop.
- You orchestrate and review; you do not implement. Hand code work to Claude
  Code via a HandOff.

## References

- `references/phase-map.md` — the canonical phase table: owner / gate / tool.
- `references/handoff-prompt.md` — the HandOff chat-block template, per phase.
- `references/report-format.md` — what Code should print back, and how to parse it.
- `references/integration-verification.md` — the cross-repo, read-only gate method.
- `references/state-discovery.md` — how to compute the current phase from disk + the latest Report.
- `references/documentation-pass.md` — the phase-6 documentation pass, step by step.
- `references/architecture-doc-template.md` — shapes for `architecture.md`, subsystem references, and ADRs.
- `references/readme-lifecycle-amendment.md` — optional in-repo text describing the lifecycle.
