# Repo-local specs and docs (`repo-local-docs`) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship `v0.1.2` of both DigiTeam plugins. In this release every spec, plan, architecture doc and ADR written into a repo describes only that repo (D1). Specs are per repo, and multi-repo features get a feature record in a primary repo (D2–D5). A new gate, "specs consistent", closes phase 0a (D6). The HandOff loses `Your scope in this repo` (D7). The architecture template gains Contracts & integrations (D8). The documentation pass runs once per repo, from the working tree (D9–D10). The executor's phase-7 check also covers self-containment and the doc hashes (D11).

**Architecture:** There is no application code. The deliverable is Markdown skill text in the two plugins plus five manifest `version` values. Code writes the skill text in phase 2 from this plan. **This plan carries the exact before → after text for every edit**, so a reviewer can check each sentence against the spec. The "tests" are greps, a parity script, a decision-anchor script (one anchor sentence per D1–D12, the evidence for spec §5.3), `claude plugin validate`, and in phase 4 the remote install round trip. Each check runs first against the current tree, where it must show the old state, and again after the edit, where it must pass.

**Tech Stack:** Markdown and JSON manifests (Claude plugin/marketplace schema). `claude` CLI 2.1.118 (`plugin validate|marketplace add|remove|install|uninstall|list`). `python` 3 for the parity and anchor scripts, reading UTF-8 with `newline=''` so LF line endings survive. `git`, and `gh` authenticated as `oleksandr-ieremchuk`.

**Spec:** [`design_docs/repo-local-docs-design.md`](../../design_docs/repo-local-docs-design.md) (committed with this plan in phase 0b, sha256 `cc01881543ba66d9dd2817ee6d58ed4d1ed6eefcd8ad0d8ff40f66dbba253a11`).

## Global Constraints

- **Release version `0.1.2`**, exactly five `version` occurrences: `metadata.version` and both plugin entries in `.claude-plugin/marketplace.json`, plus each `plugin.json`. Tag plain `v0.1.2` on the verified commit, in phase 4 only (spec D13, §5.6, §5.8). Do not use `claude plugin tag`.
- **The Report block is untouched.** Both `report-format.md` files stay byte-identical to `v0.1.1` (spec §3.1 "Unchanged on purpose", §5.2). Their sha256 today, equal to `git show v0.1.1:…`:
  - executor `9927f86548bb8af9da820233ad5267082f42fed129ec6e46ab0159edc21cb45f`
  - orchestrator `8f43623f553fe1384a1266a2f335f32bcb7e823c23869bccac486256df6fa2ae`
- **Parity rule:** both HandOff blocks change together. They open with `/delivery-executor:executing-delivery-handoff` and list the same fields in the same order. `Fix:` is the documented executor-only exception (spec D7, §5.2).
- **No `chatrevenue` / `sasha` anywhere under `plugins/`** (spec §5.7).
- **Skill text never cites spec decision numbers** (`D1`, `D9` …). Those numbers belong to this spec, and the repo-local rule applies to the skills too. The anchors in the decision table below are plain sentences.
- **LF line endings** everywhere (ADR 0006, `.gitattributes`). Edit with the Edit tool or Python `newline=''`, never a tool that rewrites CRLF.
- **Nothing outside this repository** is read or written by any step. The only external surfaces are this repo's own GitHub remote (`origin`) and the local `claude` plugin install, and phase 4 cleans that install up.
- **`design_docs/delivery-branch-and-pause-design.md` stays untracked and untouched.** It is parked Feature 3 (spec D13, §3.3) and is the one expected `??` line in every `git status`. Always `git add` explicit paths, never `git add -A` / `.`.
- **Out of scope:** the phase-6 docs of spec §3.2 (`design_docs/architecture/**`, new ADRs). Cowork authors them later, so no task here touches `design_docs/architecture/`. Also out of scope: Feature 3, the Report shape, stage folders and phase numbering (§3.3).
- **No push, no tag, no GitHub before Cowork's phase-3 deploy-approval gate.**

## Review Focus

Failure modes the spec implies but no single edit exercises. Each one is pinned by a check in Task 4.

1. **A leftover scope field or scope wording.** A stray `scope for that repo` or `this repo's scope` would reintroduce the field D7 removes. Task 4 Step 1 greps for `Your scope in this repo` and for the wider `scope for that repo|this repo's scope|and the per-repo scope|+ per-repo scope|paired spec`. Those are the old 0a phrasings; the feature record's new "per-repo scope summary" is intended and must not match.
2. **HandOff shapes drifting apart.** The template gets edited and the executor shape does not, or the field order or `Spec:` placeholder differs. Task 4 Step 2's parity script compares the field lists and the `Spec:` lines.
3. **Report block touched by accident** while editing neighbouring files. Task 4 Step 2 checks both hashes against `v0.1.1`.
4. **Decision numbers or cross-repo paths leaking into skill text.** The skill would then break the rule it teaches. Task 4 Step 5 greps `plugins/` for `\bD[0-9]{1,2}\b` and `plugins/*/skills/` for `\.\./\.\./` paths. The plugin READMEs' `../../README.md` links stay inside this repo and are exempt.
5. **Broken Markdown in the architecture template.** A nested fence closed early would break the rendered template. Task 4 Step 3 counts fences per file (must be even) and checks that each new section heading sits inside the file.

## Decision → file + section (spec §5.3; gate 3 checks this list)

Each row names the file and section that carries the decision, plus one **anchor**: an exact phrase that must appear in that file once whitespace is collapsed. Task 4 Step 4 runs the anchors as a script.

Paths abbreviate `plugins/delivery-orchestrator/skills/orchestrating-delivery/` to **O/** and `plugins/delivery-executor/skills/executing-delivery-handoff/` to **E/**.

| # | Decision | File → section | Anchor (must appear) |
|---|---|---|---|
| D1 | Repo-local rule | O/`SKILL.md` → "The division of labour (hard)" + "Hard rules"; O/`references/phase-map.md` → Notes; O/`references/documentation-pass.md` → Principles | `appear only by name, as the counterparty of a contract` |
| D2 | Specs are per repo | O/`SKILL.md` → "Routing per phase" (0a); O/`references/phase-map.md` → row 0a; O/`references/architecture-doc-template.md` → "Per-repo spec"; `README.md` → process table row 0a + "Repo conventions" | `one spec per target repo` |
| D3 | Primary repo + `<slug>-feature.md` | O/`SKILL.md` → "Routing per phase" (0a); O/`references/architecture-doc-template.md` → "Feature record"; O/`references/readme-lifecycle-amendment.md` → lifecycle block | `the only file allowed to talk about several repos` |
| D4 | Single-repo: design file doubles as feature record | O/`SKILL.md` → "Routing per phase" (0a); O/`references/state-discovery.md` → Inputs | `doubles as the feature record` |
| D5 | Discovery by feature file; ask on 0 / 2+ | O/`references/state-discovery.md` → Inputs; O/`SKILL.md` → "Locating the current phase" | `ask the user; never guess` |
| D6 | Gate "specs consistent" | O/`SKILL.md` → "Routing per phase" + "The phases and gates"; O/`references/phase-map.md` → row 0a Gate column; O/`references/state-discovery.md` → "Between 0a and 0b"; `README.md` → row 0a | `specs consistent` + `the identical shape` |
| D7 | HandOff: `Spec:` = this repo's spec, no scope field | O/`references/handoff-prompt.md` → intro + Template; E/`references/handoff-format.md` → Shape + Field table; E/`SKILL.md` → Step 1, Step 2.2; O/`SKILL.md` → HandOff bullet + "Multi-repo fan-out" | `this repo's own spec` (both HandOff files) + absence of `Your scope in this repo` |
| D8 | Contracts & integrations, Provides / Consumes, six columns | O/`references/architecture-doc-template.md` → `architecture.md` block + subsystem `Contracts` + ADR `Source:` | `## Contracts & integrations` + `\| Counterparty \| Kind / protocol \| Shape \| Auth \| Errors \| Versioning \|` |
| D9 | Documentation pass per repo, inputs = this repo only | O/`references/documentation-pass.md` → Principles + Step 3 + Step 4 (ADR `Source:`); O/`references/phase-map.md` → row 6; O/`SKILL.md` → "Routing per phase" (6) | `once per target repo` + `` never `<slug>-feature.md` `` |
| D10 | Working tree, existing draft, ADR number, sha256 list | O/`references/documentation-pass.md` → Principles + Step 0 + Step 4 + Step 5 + Step 6; O/`references/handoff-prompt.md` → phase-7 line | `in the repo's working tree` + `existing uncommitted draft` + `with its sha256` |
| D11 | Executor phase 7: self-contained, contracts match, unlisted change → blocked | E/`SKILL.md` → Step 3 phase 7 + Hard rules; E/`references/phase-map.md` → row 7 | `no path or link into another repository` + `Contracts & integrations` (both E files) |
| D12 | Gate 1 per-repo; phase 5 contracts from matrix + Provides/Consumes | O/`references/phase-map.md` → row 1; O/`references/integration-verification.md` → Method step 1 | `against its own repo's spec` + `contract matrix` |

Also carried, but not part of §5.3: **D13** (version `0.1.2`) in Task 3, and **D14** (dogfood the template) in phase 6. D14 belongs to Cowork, not this plan.

## File structure

| File | Responsibility | Task |
|---|---|---|
| O/`SKILL.md` | orchestrator entry: division of labour, routing, fan-out, hard rules | 1 |
| O/`references/phase-map.md` | canonical phase table (0a, 1, 6 rows; notes) | 1 |
| O/`references/state-discovery.md` | feature discovery + per-repo phase | 1 |
| O/`references/integration-verification.md` | phase-5 method, step 1 | 1 |
| O/`references/handoff-prompt.md` | the HandOff template (one side of parity) | 1 |
| O/`references/documentation-pass.md` | the phase-6 pass | 1 |
| O/`references/architecture-doc-template.md` | architecture, spec and feature-record shapes | 1 |
| O/`references/readme-lifecycle-amendment.md` | optional in-repo lifecycle text | 1 |
| E/`SKILL.md` | executor entry: parse, preconditions, phase 7, hard rules | 2 |
| E/`references/handoff-format.md` | the HandOff shape (other side of parity) | 2 |
| E/`references/phase-map.md` | executor phase table, row 7 | 2 |
| `README.md` | row 0a + `design_docs/` convention | 3 |
| `.claude-plugin/marketplace.json`, `plugins/*/.claude-plugin/plugin.json` | five `version` values | 3 |
| `engineering_plans/{drafts,ongoing,done}/repo-local-docs-plan.md` | this plan; the phase signal | 1 (→ ongoing), 6 (→ done) |

**Deliberately unchanged**, so no reviewer reads the omission as a miss:
- Both `report-format.md` files (spec §3.1).
- `plugins/*/README.md`. Their only "scope" hit is "never widens scope", which is not the HandOff field.
- E/`SKILL.md` "Stay in scope" hard rule. It still reads correctly: "what the spec assigns to this repo".
- E/`references/phase-map.md` row 0b "scoped to this repo". That is correct under D2.
- Everything under `design_docs/architecture/` (phase 6, Cowork).

## Editing convention

Every edit below is given as **Before** and **After** blocks. Before is the exact current text. After replaces it verbatim. Apply each with the Edit tool (`old_string` = Before, `new_string` = After). If a Before block does not match exactly, stop: the file changed since this plan was written, so report it rather than improvising. The quoted blocks are four-backtick fenced so the files' own triple-backtick fences survive.

## Phase discipline

- **Phase 2 = Tasks 1–4.** Task 1 Step 1 moves this plan `drafts → ongoing` in its own commit, then the content is edited and statically verified. There is no push and no tag. The plan ends phase 2 in `ongoing/`, and Cowork's gate 3 (change review + deploy approval) is next.
- **Phase 4 = Tasks 5–6.** This phase runs only after Cowork approves the deploy. Push `main`, run the remote install round trip at `0.1.2`, and tag `v0.1.2` on the verified commit. Then `git mv` the plan `ongoing → done` and commit.

## Who performs each verification

Each check is marked **[Code]** (Claude Code runs it in this repo and pastes the output into the Report) or **[Human]**. Every check in this plan is **[Code]**. The one human-facing step is the real reinstall in Cowork and Claude Code after release, which spec §6 / phase 5 assigns to Cowork and Sasha. It is not a step of this plan.

---

# PHASE 2 — edit and verify (Tasks 1–4)

### Task 1: Orchestrator skill (8 files)

**Files:**
- Move: `engineering_plans/drafts/repo-local-docs-plan.md` → `engineering_plans/ongoing/repo-local-docs-plan.md`
- Modify: O/`SKILL.md`, O/`references/phase-map.md`, O/`references/state-discovery.md`, O/`references/integration-verification.md`, O/`references/handoff-prompt.md`, O/`references/documentation-pass.md`, O/`references/architecture-doc-template.md`, O/`references/readme-lifecycle-amendment.md`
- Test: the pre-check grep (Step 2), the post-check grep (Step 11), the anchor script (Task 4)

**Interfaces:**
- Produces: the HandOff template's `Spec:` line `Spec: <design_docs/<feature-slug>-design.md — this repo's own spec>`. Task 2 must copy it character for character into the executor shape. It also produces the field order `Spec, Phase to run, Gate already passed, Do, Acceptance criteria for this repo, Report back`, and the section names `Contracts & integrations`, `Provides`, `Consumes`, which Task 2's phase-7 text refers to.

- [ ] **Step 1: Move the plan `drafts → ongoing` and commit that move on its own** [Code]

```bash
git mv engineering_plans/drafts/repo-local-docs-plan.md engineering_plans/ongoing/repo-local-docs-plan.md
git commit -m "$(printf '%s\n' 'plan(repo-local-docs): drafts → ongoing' '' 'Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>')"
git show --stat HEAD
```

Expected: `rename engineering_plans/{drafts => ongoing}/repo-local-docs-plan.md`, one file.

- [ ] **Step 2: Run the pre-check. The old wording must be present.** [Code]

```bash
O=plugins/delivery-orchestrator/skills/orchestrating-delivery
grep -rn "Your scope in this repo\|the scope for that repo\|this repo's scope\|and the per-repo scope\|+ per-repo scope\|paired spec" $O
grep -rln "Contracts & integrations\|specs consistent\|feature record" $O ; echo "new-terms exit=$?"
```

Expected: hits in `SKILL.md` (lines 44, 87), `references/handoff-prompt.md` (28; the 0b line at 53–54 wraps, so grep misses it, and Step 7 fixes it regardless), `references/phase-map.md` (10), `references/documentation-pass.md` (64) and `references/readme-lifecycle-amendment.md` (34). The second grep prints nothing and `new-terms exit=1`.

- [ ] **Step 3: O/`SKILL.md`**

**3a. Division of labour: state the repo-local rule (D1).**

Before:
````
- **Cowork (you)** reason about the feature **across all its repositories**: the
  spec, the review gates, cross-repo integration verification, and the
  documentation prose. You read any repo read-only and you author *content*
  (spec, architecture docs) as files for Claude Code to commit.
````
After:
````
- **Cowork (you)** reason about the feature **across all its repositories**: the
  specs, the review gates, cross-repo integration verification, and the
  documentation prose. You read any repo read-only and you author *content*
  (specs, architecture docs) as files for Claude Code to commit.
- **Repo-local rule.** Everything written into a repo — its spec, plan,
  architecture docs and ADRs — describes only that repo. Other repos and systems
  appear only by name, as the counterparty of a contract, with the contract's
  shape written out in the file. No path or link into another repo, ever. The
  single exception is the feature record, `<slug>-feature.md`, in the primary
  repo (see "Routing per phase").
````

**3b. HandOff bullet: drop the scope and point at this repo's own spec (D7).**

Before:
````
- **HandOff (you → Code):** a copy-paste chat block, one per repo, that tells
  Claude Code which repo, which spec, the scope for that repo, the phase to
  execute, the gate already passed, and the acceptance criteria. Its first line
````
After:
````
- **HandOff (you → Code):** a copy-paste chat block, one per repo, that tells
  Claude Code which repo, that repo's own spec (its `<slug>-design.md`, which
  is the whole scope), the phase to execute, the gate already passed, and the
  acceptance criteria. Its first line
````

**3c. Locating the current phase: discovery (D5).**

Before:
````
A feature is identified by a **slug** shared across all its repos (declared in
the spec, else the spec filename). To place it, inspect each target repo per
`references/state-discovery.md`: is the spec present? is there a plan for the
slug, and in which stage folder (`drafts` / `ongoing` / `done` / `documented`)?
````
After:
````
A feature is identified by a **slug** shared across all its repos (declared in
the spec, else the spec filename). Find its target repos per
`references/state-discovery.md`: from the primary repo's `<slug>-feature.md`,
or, with no feature file, from the one repo holding `<slug>-design.md`. If
that is ambiguous, ask the user; never guess. Then inspect each target repo: is
its own `<slug>-design.md` present? is there a plan for the slug, and in which
stage folder (`drafts` / `ongoing` / `done` / `documented`)?
````

**3d. The phases and gates: name the new gate (D6).**

Before:
````
`references/phase-map.md`. In short: brainstorm + spec (you) → plan (Code, per
repo) → **you review & approve the plan** → execute (Code) → **you review changes
````
After:
````
`references/phase-map.md`. In short: brainstorm + per-repo specs (you) → **you
check the specs consistent** → plan (Code, per repo) → **you review & approve
the plan** → execute (Code) → **you review changes
````

**3e. Routing per phase: 0a, the gate, and phase 6 (D2, D3, D4, D6, D9).**

Before:
````
- Phase 0a (spec) → use `superpowers:brainstorming`, then write the spec to
  `design_docs/`. The spec must list the target repos and the per-repo scope.
````
After:
````
- Phase 0a (spec) → use `superpowers:brainstorming`, then write one spec per
  target repo: `design_docs/<slug>-design.md` in that repo, describing only the
  changes to make there, with its own Provides / Consumes contracts and
  acceptance criteria (shape in `references/architecture-doc-template.md`).
  - **Multi-repo feature:** choose a **primary repo**. Propose the one holding
    the entry point or the core value, and the user confirms. Write
    `design_docs/<slug>-feature.md` there: goal, target repos, per-repo scope
    summary, cross-repo contract matrix, shared decisions, integration-
    verification plan. It is the only file allowed to talk about several repos.
    It is not folded into the primary repo's architecture docs, and no per-repo
    spec links to it.
  - **Single-repo feature:** only `<slug>-design.md`. That repo is the primary
    by definition, and its design file doubles as the feature record.
- Gate **specs consistent** (closes 0a, before any 0b HandOff) → check that
  every row of the contract matrix appears, with the identical shape, in the
  producer's Provides and the consumer's Consumes, and that every per-repo spec
  is self-contained (repo-local rule). Single-repo: self-containment only. A
  failure means fixing the specs; no HandOff goes out until it passes.
````

Before:
````
- Phase 6 (documentation) → run the documentation pass in
  `references/documentation-pass.md`. You author the architecture prose and ADRs;
  Code reviews them (phase 7) and does the `done → documented` move (phase 8).
````
After:
````
- Phase 6 (documentation) → run the documentation pass in
  `references/documentation-pass.md` once per target repo, from that repo's own
  spec, plan and code. You author the architecture prose and ADRs;
  Code reviews them (phase 7) and does the `done → documented` move (phase 8).
````

**3f. Multi-repo fan-out: each HandOff points at its own spec (D7).**

Before:
````
One spec fans out into one HandOff per target repo, all tied by the slug.
````
After:
````
One feature fans out into one HandOff per target repo, all tied by the slug.
Each HandOff points at that repo's own `<slug>-design.md`, never at the
feature record or at another repo's spec.
````

**3g. Hard rules: add the repo-local rule (D1).**

Before:
````
- You orchestrate and review; you do not implement. Hand code work to Claude
  Code via a HandOff.
````
After:
````
- You orchestrate and review; you do not implement. Hand code work to Claude
  Code via a HandOff.
- Repo-local: never write into a repo a path or link into another repo. Only the
  primary repo's `<slug>-feature.md` names several repos.
````

**3h. References list: name the new templates.**

Before:
````
- `references/architecture-doc-template.md` — shapes for `architecture.md`, subsystem references, and ADRs.
````
After:
````
- `references/architecture-doc-template.md` — shapes for `architecture.md`, subsystem references, ADRs, the per-repo spec, and the feature record.
````

- [ ] **Step 4: O/`references/phase-map.md` (D1, D2, D6, D9, D12)**

Before:
````
| 0a | Brainstorm + spec (across all target repos) | Cowork | `superpowers:brainstorming` → spec in `design_docs/` (lists target repos + per-repo scope) | — |
````
After:
````
| 0a | Brainstorm + specs | Cowork | `superpowers:brainstorming` → one `design_docs/<slug>-design.md` per target repo; multi-repo: plus `design_docs/<slug>-feature.md` in the primary repo | **specs consistent** |
````

Before:
````
| 1 | Plan review | Cowork | read each draft plan; check it against the spec | **plan approved** |
````
After:
````
| 1 | Plan review | Cowork | read each draft plan; check it against its own repo's spec only; reject any path into another repo | **plan approved** |
````

Before:
````
| 6 | Documentation | Cowork | documentation pass (`references/documentation-pass.md`), prose only, no commit | — |
````
After:
````
| 6 | Documentation | Cowork | documentation pass (`references/documentation-pass.md`) per repo, from that repo's spec + plan + code; prose only, no commit | — |
````

Before:
````
- **Gate failure → corrective HandOff, no advance.** If a Cowork-owned gate
  (1, 3, 5) fails, describe the delta in a HandOff to the relevant repo, wait for
  a new Report, and re-check the same gate. Looping here is normal.
````
After:
````
- **Gate failure → corrective HandOff, no advance.** If a Cowork-owned gate
  (1, 3, 5) fails, describe the delta in a HandOff to the relevant repo, wait for
  a new Report, and re-check the same gate. Looping here is normal. A failed
  **specs consistent** gate is fixed in the specs themselves, before any HandOff.
- **Repo-local.** Everything written into a repo describes only that repo; other
  repos and systems appear only by name, as the counterparty of a contract, with
  the shape written out. Only the primary repo's `<slug>-feature.md` talks about
  several repos.
````

- [ ] **Step 5: O/`references/state-discovery.md` (D4, D5, D6)**

Before:
````
- The **feature slug** — declared in the spec, else the spec's filename slug.
  Every artifact for the feature carries it.
- For each **target repo** (listed in the spec):
  - Is the spec present in `design_docs/` (or the repo's design home)?
  - Is there a plan for the slug, and in which stage folder:
    `drafts` / `ongoing` / `done` / `documented`?
````
After:
````
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
````

Before:
````
| spec exists, no plan | 0b — awaiting plan (send/await HandOff) |
````
After:
````
| this repo's `<slug>-design.md` exists, no plan | 0b — awaiting plan (send/await HandOff), once **specs consistent** has passed |
````

Before:
````
## Feature phase = the aggregate
````
After:
````
## Between 0a and 0b — the gate "specs consistent"

Specs on disk but no plan anywhere, and the gate **specs consistent** not yet
passed → the feature is still in 0a. Run that gate: every contract-matrix row
appears, with the identical shape, in the producer's Provides and the
consumer's Consumes, and every per-repo spec is self-contained. Single-repo:
self-containment only. Send no 0b HandOff until it passes.

## Feature phase = the aggregate
````

- [ ] **Step 6: O/`references/integration-verification.md` (D12)**

Before:
````
1. **Derive the cross-repo contracts from the spec.** List every place where one
   repo depends on something another repo shipped: tool/function signatures,
   API or message shapes, shared version numbers or guide versions, file/path
   conventions, config or env contracts.
````
After:
````
1. **Derive the cross-repo contracts** from the contract matrix in the primary
   repo's `<slug>-feature.md` plus each target repo's Provides / Consumes in its
   own `<slug>-design.md`. List every place where one repo depends on something
   another repo shipped: tool/function signatures, API or message shapes, shared
   version numbers or guide versions, file/path conventions, config or env
   contracts. The gate "specs consistent" checked that the shapes agreed on
   paper; this step checks that the shipped code agrees. For a single-repo
   feature, the contracts are that spec's Provides / Consumes against its
   external counterparties.
````

- [ ] **Step 7: O/`references/handoff-prompt.md` (D7, D10)**

Before:
````
Keep it a pointer, not a re-derivation of the spec: Code reads the spec and the
plan itself. State the phase to run, the gate already passed, and the per-repo
acceptance criteria, then name the `superpowers` skill to use.
````
After:
````
Keep it a pointer, not a re-derivation of the spec: Code reads the spec and the
plan itself. The spec is this repo's own `<slug>-design.md` and is the whole
scope, so there is no separate scope field, and the HandOff never names
another repo's files. State the phase to run, the gate already passed, and the
acceptance criteria for this repo, then name the `superpowers` skill to use.
````

Before:
````
Spec: <path to design_docs/...-design.md in this repo>
Your scope in this repo: <the slice of the feature this repo owns>
````
After:
````
Spec: <design_docs/<feature-slug>-design.md — this repo's own spec>
````

Before:
````
- **0b (plan):** "Read the spec, write the implementation plan for this repo's
  scope with `superpowers:writing-plans`, leave it in `drafts/`."
````
After:
````
- **0b (plan):** "Read the spec, write the implementation plan for this repo
  with `superpowers:writing-plans`, leave it in `drafts/`."
````

Before:
````
- **7 (review docs):** "Review the architecture-doc changes Cowork authored;
  confirm they match the real code; approve or list mismatches."
````
After:
````
- **7 (review docs):** "Review the architecture-doc changes Cowork authored in
  the working tree — these files, each with its sha256: <path — sha256, one per
  line>. Confirm they match the real code, contain no path or link into another
  repo, and that Contracts & integrations matches the real interfaces; approve
  or list mismatches."
````

- [ ] **Step 8: O/`references/documentation-pass.md` (D1, D9, D10)**

**8a. Principles.**

Before:
````
- **Discover, don't hardcode.** Find paths and the subsystem list by inspecting
  the repo, so this works in any repository.
````
After:
````
- **Discover, don't hardcode.** Find paths and the subsystem list by inspecting
  the repo, so this works in whichever repository it is run for.
- **Repo-local.** Everything you write describes only this repo. Other repos and
  systems appear only by name, as the counterparty of a contract, with the
  contract's shape written out in the file. No path or link into another repo.
- **One pass per repo.** Run the pass once per target repo. Its inputs are that
  repo's `<slug>-design.md`, its plan and its code — never `<slug>-feature.md`
  and never the cross-repo picture from phase 5.
- **Working tree, not remote.** Author in the repo's working tree and read the
  design home from there first, never from a remote or cached copy. Continue an
  existing uncommitted draft for the slug rather than writing a parallel one.
````

**8b. Step 0: the working tree and the existing-draft check.**

Before:
````
`architecture/decisions/`; the plan lifecycle folders
(`engineering_plans/{drafts,ongoing,done}` + the terminal `documented/`). Check
whether the convention already exists. State what you found before changing
anything.
````
After:
````
`architecture/decisions/`; the plan lifecycle folders
(`engineering_plans/{drafts,ongoing,done}` + the terminal `documented/`). Check
whether the convention already exists.

Work in the repo's working tree: read the design home there, not from a remote
copy. Before writing anything, look for an existing uncommitted draft for this
slug (architecture files changed but not committed, or an ADR covering the
plan's decisions). If there is one, continue it instead of writing a parallel
version. State what you found before changing anything.
````

**8c. Step 3: inputs are this repo's only.**

Before:
````
For each undocumented plan, read **the plan and its paired spec**. Distil:
````
After:
````
For each undocumented plan, read **this repo's plan, this repo's
`<slug>-design.md` and its code**. Never read `<slug>-feature.md` or another
repo's files for this. Distil:
````

**8d. Step 4: Contracts & integrations, ADR `Source:`, ADR number from disk.**

Before:
````
place. Record each newly-shipped material decision as an ADR at
`architecture/decisions/NNNN-<slug>.md` (next free number, append-only); a
````
After:
````
place. Keep `architecture.md`'s **Contracts & integrations** section (Provides /
Consumes) in sync: every interface this repo now offers or depends on has its
row, with the shape written out. Record each newly-shipped material decision as
an ADR at `architecture/decisions/NNNN-<slug>.md` (next free number, counted in
the working tree including uncommitted ADRs; append-only), with `Source:`
naming a path in this repo; a
````

**8e. The done-test.**

Before:
````
non-obvious choices were made — without reading the plan.
````
After:
````
non-obvious choices were made — without reading the plan and without opening
any other repository.
````

**8f. Step 5: cross-repo-link and ADR-number checks.**

Before:
````
every ADR has a status, every `superseded` points forward; no broken internal
links. Fix mismatches.
````
After:
````
every ADR has a status, every `superseded` points forward; no broken internal
links; no path or link into another repo in any touched file; every ADR number
is unique on disk in the working tree (no two files share `NNNN`). Fix
mismatches.
````

**8g. Step 6: the phase-7 HandOff lists files with their sha256.**

Before:
````
exact list of plans that will advance `done → documented`. This is the doc-review
handoff: build a HandOff (see `handoff-prompt.md`, phase 7) asking Code to
confirm the docs match the real code.
````
After:
````
exact list of plans that will advance `done → documented`. This is the doc-review
handoff: build a HandOff (see `handoff-prompt.md`, phase 7) that lists every
file you authored or changed, each with its sha256, and asks Code to confirm the
docs match the real code.
````

- [ ] **Step 9: O/`references/architecture-doc-template.md` (D2, D3, D8)**

**9a. Intro: name the two new templates.**

Before:
````
per-subsystem detail one click away, plus an append-only decision log.
````
After:
````
per-subsystem detail one click away, plus an append-only decision log. The
same file holds the per-repo spec template and the feature-record template.

Everything here is repo-local: it describes only this repo. Other repos and
systems appear only by name, as a contract counterparty, with the shape written
out. Never a path or link into another repo.
````

**9b. `architecture.md` block: insert Contracts & integrations before the Reference manifest.**

Before:
````
cross-cutting concerns (scheduling, observability).

## Reference manifest
````
After:
````
cross-cutting concerns (scheduling, observability).

## Contracts & integrations

What this repo offers others and what it depends on. Counterparties are named,
never linked into another repo; the shape is written out here or in a file in
this repo.

### Provides

| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |
|---|---|---|---|---|---|
| <who uses it> | <API / plugin / file / message> | <inline, or a path in this repo> | <how callers authenticate, or none> | <error shape and failure modes> | <how it is versioned> |

### Consumes

| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |
|---|---|---|---|---|---|
| <what this repo depends on> | <API / plugin / file / message> | <inline, or a path in this repo> | <credential this repo uses, or none> | <how this repo handles failure> | <version it pins or expects> |

## Reference manifest
````

**9c. Subsystem reference: split `Contracts`.**

Before:
````
## Contracts
Public surfaces other parts depend on — APIs, tool signatures, data/store shapes,
message formats. Paste the real shapes; these are load-bearing.
````
After:
````
## Contracts
Split as in `architecture.md`, with the same columns (counterparty,
kind/protocol, shape, auth, errors, versioning):
- **Provides** — public surfaces others depend on: APIs, tool signatures,
  data/store shapes, message formats.
- **Consumes** — what this subsystem depends on, inside or outside this repo.
Paste the real shapes; these are load-bearing.
````

**9d. ADR `Source:`.**

Before:
````
- **Source:** <link to the plan/spec that implemented this>
````
After:
````
- **Source:** <path in this repo to the plan or spec that implemented this>
````

**9e. Append the per-repo spec and feature-record templates at the end of the file**, after the Granularity section's last line (`ADRs, by contrast, are **per decision** — one shipped material choice each.`).

Before:
````
ADRs, by contrast, are **per decision** — one shipped material choice each.
````
After:
````
ADRs, by contrast, are **per decision** — one shipped material choice each.

## Per-repo spec — `design_docs/<slug>-design.md`

Every target repo gets its own spec, describing only the changes to make in
that repo. It follows the repo-local rule: other repos appear by name, as the
counterparty of a contract, with the shape written out.

```markdown
# <Title> — v<X.Y.Z>

- **Slug / Status / Author / Release**

## 1. Problem
As it shows up in this repo.

## 2. Decisions
| # | Decision | Alternative rejected | Why |
|---|---|---|---|

## 3. Changes in this repo
What changes, file by file or area by area. End with **Out of scope**.

## 4. Contracts
**Provides** and **Consumes**, each a table with the shape written out
(columns as in `architecture.md` Contracts & integrations).

## 5. Acceptance criteria
Numbered, each checkable in this repo.

## 6. Integration verification
Single-repo feature only. For a multi-repo feature it lives in
`<slug>-feature.md`.
```

## Feature record — `design_docs/<slug>-feature.md` (primary repo only)

Only for a multi-repo feature, and only in its primary repo. This is the one
file allowed to talk about several repos. It is not folded into the primary
repo's architecture docs, and no per-repo spec links to it. A single-repo
feature has no feature record: its `<slug>-design.md` doubles as one.

```markdown
# <Title> — feature record — v<X.Y.Z>

- **Slug / Status / Author / Release / Primary repo**

## Goal
One paragraph: what the feature achieves end to end.

## Target repos
One line each: repo name and its role in the feature.

## Per-repo scope
One short paragraph per repo; the detail lives in that repo's own spec.

## Contract matrix
| Producer | Consumer | Contract | Shape |
|---|---|---|---|

Each row must appear, with the identical shape, in the producer's Provides and
the consumer's Consumes (gate "specs consistent").

## Shared decisions
Decisions that bind more than one repo.
| # | Decision | Alternative rejected | Why |
|---|---|---|---|

## Integration-verification plan
What phase 5 exercises, per contract-matrix row.
```
````

- [ ] **Step 10: O/`references/readme-lifecycle-amendment.md` (D2, D3, D9)**

Before:
````
    -> design_docs/<topic>-design.md            (spec)
````
After:
````
    -> design_docs/<slug>-design.md             (this repo's spec; multi-repo: plus <slug>-feature.md in the primary repo)
````

Before:
````
`done/` not yet in `documented/` (plus its paired spec), fold the net
````
After:
````
`done/` not yet in `documented/` (plus this repo's `<slug>-design.md`), fold the net
````

- [ ] **Step 11: Run the post-check. Old wording gone, new terms present, fences balanced.** [Code]

```bash
O=plugins/delivery-orchestrator/skills/orchestrating-delivery
grep -rn "Your scope in this repo\|the scope for that repo\|this repo's scope\|and the per-repo scope\|+ per-repo scope\|paired spec" $O ; echo "old-terms exit=$?"
grep -rlc "specs consistent" $O
python - <<'EOF'
import glob
for p in sorted(glob.glob('plugins/delivery-orchestrator/skills/orchestrating-delivery/**/*.md', recursive=True)):
    n = sum(1 for l in open(p, encoding='utf-8') if l.lstrip().startswith('```'))
    assert n % 2 == 0, (p, n)
    print('fences OK', n, p)
EOF
git diff --stat
```

Expected: `old-terms exit=1`. `specs consistent` is listed in `SKILL.md`, `phase-map.md`, `state-discovery.md`, `integration-verification.md` and `architecture-doc-template.md`. Every file prints `fences OK` with an even count. `git diff --stat` shows exactly the 8 orchestrator files and nothing else.

- [ ] **Step 12: Commit** [Code]

```bash
git add plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/phase-map.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/state-discovery.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/integration-verification.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/documentation-pass.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/architecture-doc-template.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/references/readme-lifecycle-amendment.md
git status --porcelain
git commit -m "$(printf '%s\n' 'feat(orchestrator): repo-local specs and docs' '' 'Per-repo specs plus a feature record in the primary repo; gate "specs' 'consistent" closes 0a; HandOff drops "Your scope in this repo"; architecture' 'template gains Contracts & integrations (Provides / Consumes) and the spec and' 'feature-record templates; documentation pass runs per repo from the working' 'tree and lists authored files with sha256 for phase 7.' '' 'Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>')"
```

Expected before the commit: 8 staged `M` lines, and only `?? design_docs/delivery-branch-and-pause-design.md` besides them.

---

### Task 2: Executor skill (3 files)

**Files:**
- Modify: E/`SKILL.md`, E/`references/handoff-format.md`, E/`references/phase-map.md`
- Test: the pre-/post-grep; the parity script (Task 4 Step 2)

**Interfaces:**
- Consumes from Task 1: the exact `Spec:` line `Spec: <design_docs/<feature-slug>-design.md — this repo's own spec>`, the field order, and the section name `Contracts & integrations`.
- Produces: the executor side of the HandOff parity, and the phase-7 behaviour Cowork's phase-7 HandOffs rely on (a file list with sha256, and a blocked Report on an unlisted change).

- [ ] **Step 1: Run the pre-check** [Code]

```bash
E=plugins/delivery-executor/skills/executing-delivery-handoff
grep -rn "Your scope in this repo\|this repo's scope\|\*\*scope\*\*" $E
grep -rn "Contracts & integrations\|sha256" $E ; echo "new-terms exit=$?"
```

Expected: hits in `SKILL.md` (36, 49, 67) and `references/handoff-format.md` (20, 47, 48); `new-terms exit=1`.

- [ ] **Step 2: E/`SKILL.md` (D7, D11)**

**2a. Step 1 field list.**

Before:
````
- **spec** path, **scope** in this repo, **phase to run** (0b / 2 / 4 / 7 / 8),
  **gate already passed**;
````
After:
````
- **spec** path (this repo's own spec; all of it is yours), **phase to run**
  (0b / 2 / 4 / 7 / 8), **gate already passed**;
````

**2b. Step 2.2.**

Before:
````
2. **Spec present** at the given path. Read it; find the slug and this repo's
   scope in it.
````
After:
````
2. **Spec present** at the given path. Read it and find the slug. The spec is
   this repo's own; everything in it is yours to implement. If it points at a
   path or link in another repository, do not follow it: report it under
   `Blockers` and stop.
````

**2c. Step 3, phase 0b wording.**

Before:
````
- **0b — write the plan.** Read the spec; write the implementation plan for
  *this repo's scope only* with `superpowers:writing-plans`; save it in
````
After:
````
- **0b — write the plan.** Read the spec; write the implementation plan for
  *this repo only* with `superpowers:writing-plans`; save it in
````

**2d. Step 3, phase 7.**

Before:
````
- **7 — review docs.** Cowork has authored architecture prose / ADRs. Read
  them against the real code and list every mismatch; approve only when there
  are none. No plan move in this phase.
````
After:
````
- **7 — review docs.** Cowork has authored architecture prose / ADRs in the
  working tree, and the HandOff lists each file with its sha256. First confirm
  that the files on disk match those hashes and that the design home has no
  uncommitted change the HandOff does not list. Either mismatch → a blocked
  Report. Then check three things and list every mismatch: the docs match the
  real code; they are self-contained (no path or link into another repository);
  the **Contracts & integrations** tables (Provides / Consumes) match the real
  interfaces. Approve only when there are none. No plan move in this phase.
````

**2e. Hard rules: never leave this repository.**

Before:
````
- **One repo, one phase, one HandOff.** Never touch another repository; never
  run the next phase "while you're at it"; never advance the plan stage beyond
  what the phase specifies.
````
After:
````
- **One repo, one phase, one HandOff.** Never touch another repository; never
  run the next phase "while you're at it"; never advance the plan stage beyond
  what the phase specifies.
- **Never leave this repository.** Do not read or follow a path into another
  repository, even when a spec, plan or doc mentions one. Report it under
  `Blockers` instead.
````

- [ ] **Step 3: E/`references/handoff-format.md` (D7)**

Before:
````
Spec: <path to design_docs/...-design.md in this repo>
Your scope in this repo: <the slice of the feature this repo owns>
````
After:
````
Spec: <design_docs/<feature-slug>-design.md — this repo's own spec>
````

Before:
````
| `Spec` | Read it first. Find the target-repo list and this repo's scope. |
| `Your scope in this repo` | The boundary of what you implement. Anything else in the spec belongs to another repo. |
````
After:
````
| `Spec` | This repo's own spec; everything in it is yours. Read it first. It never needs another repo — a path or link into one is a blocker. |
````

- [ ] **Step 4: E/`references/phase-map.md` row 7 (D11)**

Before:
````
| **7** review docs | `done` | read Cowork's architecture prose / ADRs against the real code; approve or list mismatches; **no move** | `done` | — (Cowork fixes docs or sends phase 8) |
````
After:
````
| **7** review docs | `done` | check the listed doc files against their sha256 and that the design home has no unlisted uncommitted change (else blocked); then read the docs against the real code, for self-containment (no path or link into another repository), and that Contracts & integrations matches the real interfaces; approve or list mismatches; **no move** | `done` | — (Cowork fixes docs or sends phase 8) |
````

- [ ] **Step 5: Run the post-check** [Code]

```bash
E=plugins/delivery-executor/skills/executing-delivery-handoff
grep -rn "Your scope in this repo\|this repo's scope\|\*\*scope\*\*" $E ; echo "old-terms exit=$?"
grep -rln "Contracts & integrations" $E
git diff --stat
```

Expected: `old-terms exit=1`. `Contracts & integrations` appears in `SKILL.md` and `references/phase-map.md`. The diff shows exactly the 3 executor files.

- [ ] **Step 6: Commit** [Code]

```bash
git add plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md \
        plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md \
        plugins/delivery-executor/skills/executing-delivery-handoff/references/phase-map.md
git commit -m "$(printf '%s\n' 'feat(executor): spec is the scope; phase 7 checks self-containment' '' 'HandOff shape drops "Your scope in this repo" in step with the orchestrator' 'template. The executor never follows a path into another repo, and phase 7' 'verifies the listed doc hashes, blocks on unlisted uncommitted changes, and' 'checks self-containment and Contracts & integrations.' '' 'Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>')"
```

---

### Task 3: Root README and the five versions (`0.1.2`)

**Files:**
- Modify: `README.md` (process table row 0a; "Repo conventions" `design_docs/` bullet)
- Modify: `.claude-plugin/marketplace.json` (3×), `plugins/delivery-orchestrator/.claude-plugin/plugin.json`, `plugins/delivery-executor/.claude-plugin/plugin.json`
- Test: the version grep, the manifest contract script, `claude plugin validate` ×3

**Interfaces:**
- Consumes: the 0a row wording from Task 1 Step 4. The README row mirrors it.
- Produces: `version == "0.1.2"` in five places, which Task 5 reads back from the remote install.

- [ ] **Step 1: README row 0a (D2, D6)**

Before:
````
| 0a | Brainstorm + spec (across all target repos) | Cowork | `superpowers:brainstorming` → spec in `design_docs/` (lists target repos + per-repo scope) | — |
````
After:
````
| 0a | Brainstorm + specs | Cowork | `superpowers:brainstorming` → one `design_docs/<slug>-design.md` per target repo; multi-repo: plus `design_docs/<slug>-feature.md` in the primary repo | **specs consistent** |
````

- [ ] **Step 2: README `design_docs/` convention (D2, D3)**

Before:
````
- `design_docs/` — one spec per feature, `<slug>-design.md`, listing the target
  repos and the per-repo scope.
````
After:
````
- `design_docs/` — one spec per feature per repo, `<slug>-design.md`,
  describing only that repo's changes and its Provides / Consumes contracts. A
  multi-repo feature also has `<slug>-feature.md` in its primary repo, the only
  file that talks about several repos.
````

- [ ] **Step 3: Run the failing version check** [Code]

```bash
grep -rn '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
```

Expected: five lines, all `0.1.1`.

- [ ] **Step 4: Bump the five occurrences** [Code]

```bash
python - <<'EOF'
for p, n in (('.claude-plugin/marketplace.json', 3),
             ('plugins/delivery-orchestrator/.claude-plugin/plugin.json', 1),
             ('plugins/delivery-executor/.claude-plugin/plugin.json', 1)):
    s = open(p, encoding='utf-8', newline='').read()
    assert s.count('"version": "0.1.1"') == n, (p, s.count('"version": "0.1.1"'))
    open(p, 'w', encoding='utf-8', newline='').write(s.replace('"version": "0.1.1"', '"version": "0.1.2"'))
    print(p, ':', n, 'replaced')
EOF
```

Expected: `3 replaced`, `1 replaced`, `1 replaced`.

- [ ] **Step 5: Re-check versions and the manifest contract** [Code]

```bash
grep -rn '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
grep -rn '0\.1\.1' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json ; echo "stale exit=$?"
python - <<'EOF'
import json, os
m = json.load(open('.claude-plugin/marketplace.json', encoding='utf-8'))
assert m['name'] == 'digiteam'
assert m['metadata']['version'] == '0.1.2'
for e in m['plugins']:
    pj = json.load(open(os.path.join(e['source'], '.claude-plugin', 'plugin.json'), encoding='utf-8'))
    assert pj['name'] == e['name'] and pj['description'] == e['description'], e['name']
    assert pj['version'] == e['version'] == '0.1.2', (pj['version'], e['version'])
    assert 'dependencies' not in pj
    print('OK', e['name'], pj['version'])
print('contract OK at 0.1.2')
EOF
```

Expected: five `0.1.2` lines, `stale exit=1`, two `OK … 0.1.2` lines and `contract OK at 0.1.2`.

- [ ] **Step 6: Validate** [Code]

```bash
claude plugin validate .
claude plugin validate ./plugins/delivery-orchestrator
claude plugin validate ./plugins/delivery-executor
```

Expected: three passes, no errors (spec §5.6).

- [ ] **Step 7: Commit** [Code]

```bash
git add README.md .claude-plugin/marketplace.json plugins/delivery-orchestrator/.claude-plugin/plugin.json plugins/delivery-executor/.claude-plugin/plugin.json
git commit -m "$(printf '%s\n' 'chore(release): per-repo spec convention in README; bump to 0.1.2' '' 'README lifecycle row 0a and the design_docs convention describe per-repo' 'specs, the feature record and the "specs consistent" gate. All five' 'version occurrences move to 0.1.2 together.' '' 'Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>')"
```

---

### Task 4: Static verification sweep (spec §5.1, §5.2, §5.3, §5.4–§5.7)

Read-only. It produces the evidence block for the Report. No commit.

- [ ] **Step 1: §5.1: `Your scope in this repo` is gone, plus the wider scope wording (Review Focus 1)** [Code]

```bash
grep -rn "Your scope in this repo" plugins/ README.md ; echo "5.1 exit=$?"
grep -rn "the scope for that repo\|this repo's scope\|and the per-repo scope\|+ per-repo scope\|paired spec" plugins/ README.md ; echo "wide exit=$?"
```

Expected: `5.1 exit=1`, `wide exit=1`.

- [ ] **Step 2: §5.2: parity, and Report blocks byte-identical to `v0.1.1` (Review Focus 2, 3)** [Code]

```bash
python - <<'EOF'
import re, hashlib, subprocess
SLASH = '/delivery-executor:executing-delivery-handoff'
O = 'plugins/delivery-orchestrator/skills/orchestrating-delivery/references/'
E = 'plugins/delivery-executor/skills/executing-delivery-handoff/references/'
def first_block(t):
    return re.search(r'^```\n(.*?)^```', t, re.S | re.M).group(1)
def disk(p): return open(p, encoding='utf-8', newline='').read()
def at(tag, p): return subprocess.run(['git', 'show', f'{tag}:{p}'], capture_output=True, text=True, encoding='utf-8', check=True).stdout

# Report files unchanged since v0.1.1, and the two Report blocks identical
for p in (O + 'report-format.md', E + 'report-format.md'):
    assert disk(p) == at('v0.1.1', p), p
    print('OK  unchanged since v0.1.1:', p, hashlib.sha256(disk(p).encode()).hexdigest()[:16])
assert first_block(disk(O + 'report-format.md')) == first_block(disk(E + 'report-format.md'))
print('OK  Report blocks identical')

# HandOff blocks: slash line, same fields in order, Fix: executor-only, identical Spec line
o, e = first_block(disk(O + 'handoff-prompt.md')), first_block(disk(E + 'handoff-format.md'))
assert o.splitlines()[0] == SLASH and e.splitlines()[0] == SLASH
fields = lambda b: [l.split(':')[0] for l in b.splitlines() if re.match(r'^[A-Z][A-Za-z ]+:', l)]
of, ef = fields(o), fields(e)
assert [f for f in ef if f != 'Fix'] == of, (of, ef)
assert 'Fix' in ef and 'Fix' not in of
assert of == ['Spec', 'Phase to run', 'Gate already passed', 'Do', 'Acceptance criteria for this repo', 'Report back'], of
spec_line = lambda b: next(l for l in b.splitlines() if l.startswith('Spec:'))
assert spec_line(o) == spec_line(e), (spec_line(o), spec_line(e))
print('OK  HandOff parity:', of, '| Spec line:', spec_line(o))
print('parity OK')
EOF
```

Expected: two `unchanged since v0.1.1` lines (hash prefixes `8f43623f553fe138` orchestrator, `9927f86548bb8af9` executor), `Report blocks identical`, the parity line, and `parity OK`.

- [ ] **Step 3: §5.4 and §5.5: template sections and documentation-pass wording (Review Focus 5)** [Code]

```bash
python - <<'EOF'
O = 'plugins/delivery-orchestrator/skills/orchestrating-delivery/references/'
norm = lambda p: ' '.join(open(p, encoding='utf-8').read().split())
t = norm(O + 'architecture-doc-template.md')
for s in ('## Contracts & integrations', '### Provides', '### Consumes',
          '| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |',
          '## Per-repo spec — `design_docs/<slug>-design.md`',
          '## Feature record — `design_docs/<slug>-feature.md` (primary repo only)',
          '| Producer | Consumer | Contract | Shape |'):
    assert s in t, s
assert t.count('| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |') == 2
print('5.4 OK  template sections present')
d = norm(O + 'documentation-pass.md')
for s in ('without opening any other repository',                 # done-test
          "Work in the repo's working tree", 'existing uncommitted draft',   # Step 0
          'no path or link into another repo in any touched file',          # Step 5
          'every ADR number is unique on disk'):                            # Step 5
    assert s in d, s
print('5.5 OK  documentation-pass wording present')
EOF
```

Expected: `5.4 OK …` and `5.5 OK …`.

- [ ] **Step 4: §5.3: decision anchors D1–D12 (the table above, run mechanically)** [Code]

```bash
python - <<'EOF'
O = 'plugins/delivery-orchestrator/skills/orchestrating-delivery/'
E = 'plugins/delivery-executor/skills/executing-delivery-handoff/'
norm = lambda p: ' '.join(open(p, encoding='utf-8').read().split())
A = {
 'D1':  [(O+'SKILL.md', 'appear only by name, as the counterparty of a contract'),
         (O+'references/phase-map.md', 'appear only by name, as the counterparty of a contract'),
         (O+'references/documentation-pass.md', 'appear only by name, as the counterparty of a contract')],
 'D2':  [(O+'SKILL.md', 'one spec per target repo'),
         (O+'references/phase-map.md', 'one `design_docs/<slug>-design.md` per target repo'),
         (O+'references/architecture-doc-template.md', 'Every target repo gets its own spec'),
         ('README.md', 'one spec per feature per repo')],
 'D3':  [(O+'SKILL.md', 'the only file allowed to talk about several repos'),
         (O+'references/architecture-doc-template.md', 'the one file allowed to talk about several repos'),
         (O+'references/readme-lifecycle-amendment.md', '<slug>-feature.md in the primary repo')],
 'D4':  [(O+'SKILL.md', 'doubles as the feature record'),
         (O+'references/state-discovery.md', 'doubles as the feature record')],
 'D5':  [(O+'references/state-discovery.md', 'ask the user; never guess'),
         (O+'SKILL.md', "ask the user; never guess")],
 'D6':  [(O+'SKILL.md', 'Gate **specs consistent** (closes 0a, before any 0b HandOff)'),
         (O+'SKILL.md', 'the identical shape'),
         (O+'references/phase-map.md', '| **specs consistent** |'),
         (O+'references/state-discovery.md', 'the identical shape'),
         ('README.md', '| **specs consistent** |')],
 'D7':  [(O+'references/handoff-prompt.md', "this repo's own spec"),
         (E+'references/handoff-format.md', "this repo's own spec"),
         (E+'SKILL.md', "this repo's own spec; all of it is yours"),
         (O+'SKILL.md', "that repo's own `<slug>-design.md`")],
 'D8':  [(O+'references/architecture-doc-template.md', '## Contracts & integrations'),
         (O+'references/architecture-doc-template.md', '| Counterparty | Kind / protocol | Shape | Auth | Errors | Versioning |'),
         (O+'references/architecture-doc-template.md', '<path in this repo to the plan or spec that implemented this>')],
 'D9':  [(O+'references/documentation-pass.md', 'once per target repo'),
         (O+'references/documentation-pass.md', 'never `<slug>-feature.md`'),
         (O+'references/documentation-pass.md', 'naming a path in this repo'),
         (O+'references/phase-map.md', "per repo, from that repo's spec + plan + code"),
         (O+'SKILL.md', 'once per target repo')],
 'D10': [(O+'references/documentation-pass.md', "in the repo's working tree"),
         (O+'references/documentation-pass.md', 'existing uncommitted draft'),
         (O+'references/documentation-pass.md', 'counted in the working tree including uncommitted ADRs'),
         (O+'references/documentation-pass.md', 'with its sha256'),
         (O+'references/handoff-prompt.md', 'each with its sha256')],
 'D11': [(E+'SKILL.md', 'no path or link into another repository'),
         (E+'SKILL.md', 'uncommitted change the HandOff does not list'),
         (E+'SKILL.md', 'Contracts & integrations'),
         (E+'SKILL.md', 'Never leave this repository'),
         (E+'references/phase-map.md', 'no path or link into another repository'),
         (E+'references/phase-map.md', 'Contracts & integrations')],
 'D12': [(O+'references/phase-map.md', "against its own repo's spec"),
         (O+'references/integration-verification.md', 'contract matrix')],
}
bad = 0
for d, rows in A.items():
    for p, s in rows:
        ok = s in norm(p)
        bad += not ok
        print('OK ' if ok else 'MISS', d, p.replace(O, 'O/').replace(E, 'E/'), '::', s)
assert not bad, f'{bad} anchors missing'
print('5.3 OK  D1–D12 traceable')
EOF
```

Expected: every line `OK`, then `5.3 OK  D1–D12 traceable`. Paste the output into the Report. It is the gate-3 evidence for §5.3.

- [ ] **Step 5: §5.7 and Review Focus 4: generic names, no decision numbers, no cross-repo paths** [Code]

```bash
grep -rin "chatrevenue\|sasha" plugins/ ; echo "5.7 exit=$?"
grep -rnE '\bD[0-9]{1,2}\b' plugins/ ; echo "dnum exit=$?"
grep -rn '\.\./\.\./' plugins/*/skills/ ; echo "xrepo exit=$?"
```

Expected: `5.7 exit=1`, `dnum exit=1`, `xrepo exit=1`.

- [ ] **Step 6: §5.6: validate + versions** [Code]

```bash
claude plugin validate . && claude plugin validate ./plugins/delivery-orchestrator && claude plugin validate ./plugins/delivery-executor
grep -rn '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
```

Expected: three passes; five `0.1.2` lines.

- [ ] **Step 7: §5.9: tree, history and stage** [Code]

```bash
git status --porcelain
git log --oneline -6
ls engineering_plans/*/
```

Expected: only `?? design_docs/delivery-branch-and-pause-design.md`. Four phase-2 commits on top of the phase-0b commit (`plan … → ongoing`, `feat(orchestrator)`, `feat(executor)`, `chore(release)`). `repo-local-docs-plan.md` sits in `ongoing/` only.

**Phase 2 ends here.** Print the Report and stop. Gate 3 is next.

---

# PHASE 4 — release (Tasks 5–6), only after deploy approval

### Task 5: Push `main` and run the remote install round trip at `0.1.2` (spec §5.8)

**Interfaces:**
- Consumes: the phase-2 commits, plus Cowork's deploy approval named in the HandOff's `Gate already passed`.
- Produces: the pushed, verified commit that Task 6 tags.

- [ ] **Step 1: Starting state** [Code]

```bash
git status --porcelain
git log --oneline origin/main..main
git tag -l
```

Expected: only the untracked Feature-3 spec. Local `main` is ahead of `origin/main` by the phase-0b commit and the phase-2 commits. Tags are `v0.1.0` and `v0.1.1`, with no `v0.1.2`.

- [ ] **Step 2: Push** [Code]

```bash
git push origin main
git rev-parse main origin/main
```

Expected: both SHAs equal.

- [ ] **Step 3: Remote round trip from a clean plugin state** [Code]

```bash
claude plugin list
claude plugin marketplace list
```

If `delivery-executor` or a `digiteam` marketplace is present, remove them first:

```bash
claude plugin uninstall delivery-executor
claude plugin marketplace remove digiteam
```

Then:

```bash
claude plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace
claude plugin install delivery-executor@digiteam
claude plugin list
```

Expected: `delivery-executor` at **`0.1.2`**. If it reads `0.1.1`, a cache is serving the old manifest: remove, re-add, reinstall. Do not tag until it reads `0.1.2`.

- [ ] **Step 4: Installed content is the new content** [Code]

```bash
INSTALLED=$(find ~/.claude/plugins -path '*executing-delivery-handoff/SKILL.md' 2>/dev/null | head -1)
echo "$INSTALLED"
sha256sum "$INSTALLED" plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md
grep -c "Never leave this repository" "$INSTALLED"
gh api repos/oleksandr-ieremchuk/digiteam-cowork-marketplace/contents/.claude-plugin/marketplace.json --jq '.content' | base64 -d | grep -n '"version"'
```

Expected: the two hashes are equal, the grep prints `1`, and the published manifest shows three `0.1.2` lines.

- [ ] **Step 5: Clean up** [Code]

```bash
claude plugin uninstall delivery-executor
claude plugin marketplace remove digiteam
claude plugin list
```

Expected: no leftovers. The real reinstalls in Cowork and Claude Code belong to Cowork and Sasha at phase 5 **[Human, not in this plan]**.

---

### Task 6: Tag `v0.1.2` on the verified commit and move the plan `ongoing → done`

- [ ] **Step 1: Tag and push the tag** [Code]

```bash
git tag -a v0.1.2 -m "digiteam marketplace v0.1.2 — repo-local specs and docs"
git push origin v0.1.2
git rev-parse 'v0.1.2^{commit}' origin/main
git ls-remote --tags origin
```

Expected: `v0.1.2^{commit}` equals `origin/main`, the commit verified in Task 5, and `v0.1.2` is listed remotely.

- [ ] **Step 2: Move the plan and commit** [Code]

```bash
git mv engineering_plans/ongoing/repo-local-docs-plan.md engineering_plans/done/repo-local-docs-plan.md
git commit -m "$(printf '%s\n' 'plan(repo-local-docs): ongoing → done' '' 'Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>')"
git push origin main
```

- [ ] **Step 3: Final state** [Code]

```bash
ls engineering_plans/*/
git status --porcelain
git log --oneline -2
```

Expected: the plan is in `done/` only. Status shows only the untracked Feature-3 spec. The stage-move commit is pushed and sits after the tag, as with `v0.1.0` and `v0.1.1`: the tag marks released content, not bookkeeping.

**Phase 4 ends here.** Print the Report. Gate 5 is next. Phase 6 (the spec §3.2 docs) is Cowork's and is not part of this plan.

---

## Acceptance criteria → steps

| Spec §5 | Verified by | Who |
|---|---|---|
| 1. no `Your scope in this repo` | Task 4 Step 1 | [Code] |
| 2. HandOff parity; Report blocks byte-identical to v0.1.1 | Task 4 Step 2 | [Code] |
| 3. D1–D12 traceable to file + section | decision table above; Task 4 Step 4 | [Code] runs, Cowork gate 3 judges |
| 4. template: Contracts & integrations (both tables, D8 columns), per-repo spec, feature record | Task 1 Step 9; Task 4 Step 3 | [Code] |
| 5. documentation-pass: done-test phrase, Step 0 working tree + draft check, Step 5 link + ADR-number checks | Task 1 Step 8; Task 4 Step 3 | [Code] |
| 6. `claude plugin validate` ×3; five versions `0.1.2` | Task 3 Steps 5–6; Task 4 Step 6 | [Code] |
| 7. no `chatrevenue` / `sasha` under `plugins/` | Task 4 Step 5 | [Code] |
| 8. remote round trip at 0.1.2; tag `v0.1.2` on the verified commit | Task 5; Task 6 Step 1 | [Code] |
| 9. plan in the stage folder matching the reported phase | Task 1 Step 1 (`ongoing`); Task 4 Step 7; Task 6 Step 2 (`done`) | [Code] |

## Open risks

- **R1: Before blocks drift.** If any skill file changes between plan approval and phase 2, an Edit's Before won't match. The convention above says stop and report. Do not improvise text, because gate 3 checks against this plan.
- **R2: Remote cache serves `0.1.1`.** Task 5 Steps 3–4 compare the installed `SKILL.md` hash with the repo copy before tagging.
- **R3: Anchor script is only as good as its anchors.** It proves each decision has a sentence, not that the sentence is right. The full Before → After text in Tasks 1–3 is what gate 3 reads for meaning.
