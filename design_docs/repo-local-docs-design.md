# Repo-local specs and docs — v0.1.2

- **Slug:** `repo-local-docs`
- **Status:** spec — awaiting gate "specs consistent", then planning (phase 0b)
- **Author:** Cowork (orchestrator), 2026-09-28
- **Target repos:** `digiteam-cowork-marketplace` (this repo) — single-repo feature, so this file is also the
  feature record (no `<slug>-feature.md`, per D4)
- **Release:** `v0.1.2` — both plugins bump `0.1.1 → 0.1.2`

## 1. Problem

Bug report from use: in the documentation pass (phase 6) Cowork writes documentation about the *feature*
instead of about the *repository*, and links from one repo's docs into another repo. The 0.1.1 skill never
says otherwise, and several places push it that way:

1. The pass is not scoped per repo. It runs right after the cross-repo integration gate, with the whole
   feature in view, and `documentation-pass.md` only says it "works in any repository".
2. Its input is "the plan and its paired spec", and the spec is one cross-repo document. The same spec is
   what every repo's `Spec:` field points at, so it leaks into plans and docs everywhere.
3. The ADR template's `Source:` is "link to the plan/spec", which can be a path in another repo. Nothing
   forbids links into other repos.
4. The template's `Contracts` covers only what a subsystem *provides*. What a repo *consumes* from other
   repos and external systems has no home, and nothing requires the contract shape to be written out rather
   than linked.
5. The done-test ("a new engineer can read `architecture.md` …") and the phase-7 review never check that
   the docs stand on their own.

A second defect surfaced while finishing `executor-trigger` (2026-09-28): Cowork authored phase-6 docs from
the **remote** copy of the repo, while an earlier, uncommitted phase-6 draft already sat in the **working
tree**. The result was two ADR 0007 files under different names. Code's clean-tree check caught it at
phase 7; the skill itself should prevent it.

## 2. Decisions taken (brainstorm with Sasha, 2026-09-28)

| # | Decision | Alternative rejected | Why |
|---|---|---|---|
| D1 | **Repo-local rule.** Everything written into a repo — spec, plan, architecture docs, ADRs — describes only that repo. Other repos and systems appear only **by name**, as the counterparty of a contract, with the contract **shape written out** in the file. No paths or links into another repo. | allow cross-repo links "where convenient" | A repo's docs must be readable and true without checking out anything else; links rot and drag in other repos' context. |
| D2 | **Specs are per repo.** Every target repo gets its own `design_docs/<slug>-design.md` describing the changes to make **in that repo**, with its own Provides / Consumes contracts and acceptance criteria. | one cross-repo spec copied into each repo | The cross-repo spec is the root cause of the leakage (§1.2). |
| D3 | **Primary repo.** Each multi-repo feature has one primary repo, chosen at 0a (Cowork proposes — normally the repo holding the entry point or the core value — Sasha confirms). It additionally holds `design_docs/<slug>-feature.md`: goal, target repos, per-repo scope summary, cross-repo contract matrix, shared decisions, integration-verification plan. This is the only file allowed to talk about several repos. It is not folded into the primary repo's architecture docs, and no per-repo spec links to it. | a GitHub issue as the feature record; Cowork project docs | Not every setup has GitHub (Sasha); the record must live with the code, not in a tool. |
| D4 | **Single-repo feature:** only `<slug>-design.md`; that repo is the primary by definition and the design file doubles as the feature record. | always two files | Two near-identical files for one repo add nothing. |
| D5 | **Discovery.** Cowork locates a feature by `<slug>-feature.md` across the mounted repos and reads the target-repo list from it. No feature file and `<slug>-design.md` in exactly one repo → single-repo feature. Zero or several feature files → ask the user, never guess. | target list inside every spec | Per-repo specs must not list other repos (D1). |
| D6 | **New gate "specs consistent"** closes phase 0a, before any 0b HandOff. Checks: every contract row of the matrix appears, with the identical shape, in the producer's Provides and the consumer's Consumes; every per-repo spec passes the self-containment check (D1). Single-repo: self-containment only. | catch contract drift at phase 5 | Phase 5 finds a mismatch after code is written and deployed; this finds it before planning. |
| D7 | **HandOff:** `Spec:` is this repo's `<slug>-design.md`; the field `Your scope in this repo` is **dropped** (the spec *is* the scope). No primary-repo field — Code never needs another repo. Both HandOff copies change together (parity rule). The Report block is unchanged. | add `Primary repo:` | Code must not reach outside its repo; a field it may not use invites it to. |
| D8 | **Architecture template:** `architecture.md` gets a mandatory section **Contracts & integrations** with two tables, **Provides** and **Consumes**. Columns: counterparty, kind/protocol, shape (inline, or a file *in this repo*), auth, errors, versioning. A subsystem reference's `Contracts` splits the same way. | leave contracts inside subsystem references only | Consumed contracts had no home (§1.4). |
| D9 | **Documentation pass per repo.** One pass per target repo; inputs are that repo's `<slug>-design.md`, its plan and its code — never `<slug>-feature.md` and never the cross-repo picture from phase 5. ADR `Source:` names a path in the same repo. | one pass for the feature | §1.1–1.3. |
| D10 | **Working tree, not remote.** Cowork authors phase-6 docs **in the repo's working tree** and reads the design home from there first. Before writing, it looks for an existing uncommitted draft for this slug (architecture files or an ADR covering the plan's decisions) and continues it rather than writing a parallel version; an ADR number is "next free" only after checking the working tree. The phase-7 HandOff lists every authored file with its sha256. | author in a side folder from the remote copy | The `executor-trigger` defect (§1, last paragraph). |
| D11 | **Executor phase-7 check.** Besides "matches the code": the docs are self-contained (no path or link into another repo), and the Contracts & integrations tables match the real interfaces. Uncommitted changes in the design home not listed in the HandOff → blocked Report. | review against code only | §1.5; codifies what Code already did by instinct on 2026-09-28. |
| D12 | **Gate 1 and phase 5 follow suit.** Gate 1 checks a plan against its own repo's spec only and rejects paths into other repos. Phase 5 derives the contracts from the feature matrix plus each repo's Provides/Consumes. | — | Consistency with D2/D3. |
| D13 | **Version 0.1.2** in all five places, tag `v0.1.2`. `delivery-branch-and-pause` (Feature 3, spec untracked in the working tree) moves to 0.1.3 when resumed; its spec is amended then, not here. | 0.2.0 | Sasha's choice: versions follow release order. |
| D14 | **Dogfood.** Phase 6 of this feature brings this repo's own architecture set to the new template (adds Contracts & integrations to `architecture.md`, see §3.2). | separate task | Proves the template on a real repo in the same release. |

## 3. Changes in this repo

### 3.1 Skill content (Code writes it in phase 2 from this spec)

Orchestrator — `plugins/delivery-orchestrator/skills/orchestrating-delivery/`:

| File | Change |
|---|---|
| `SKILL.md` | Division of labour and routing: D1 stated as a hard rule; phase 0a produces per-repo specs (D2) and, for multi-repo features, the primary repo's `<slug>-feature.md` (D3/D4); the "specs consistent" gate (D6); "Multi-repo fan-out" says each HandOff points at that repo's own spec; the HandOff bullet drops "the scope for that repo". |
| `references/phase-map.md` | 0a row: artifacts per D2–D4 and gate **specs consistent**; 1 row: "against this repo's spec"; 6 row: "per repo (D9)"; notes gain D1. |
| `references/state-discovery.md` | Inputs per D5 (feature file → target list; single-repo fallback; ask on 0 / 2+). Per-repo table: "this repo's `<slug>-design.md` exists, no plan" → 0b. Gate "specs consistent" as the state between 0a and 0b. |
| `references/integration-verification.md` | Step 1 derives contracts per D12. |
| `references/handoff-prompt.md` | Template: `Spec:` = this repo's `<slug>-design.md`; `Your scope in this repo` removed (D7); 0b instruction "for this repo" wording kept; phase-7 instruction adds "the listed files and their sha256". |
| `references/documentation-pass.md` | Principles gain D1, D9, D10. Step 0: work in the working tree, look for an existing draft. Step 3 inputs per D9. Step 4: Contracts & integrations kept in sync; ADR `Source:` in-repo. Step 5 adds: no path/link into another repo; ADR number unique on disk. Step 6: HandOff lists authored files + sha256. Done-test: "… without opening any other repository". |
| `references/architecture-doc-template.md` | `architecture.md` gains **Contracts & integrations** (Provides / Consumes tables, columns per D8); subsystem `Contracts` split likewise; ADR `Source:` = path in this repo. New section: **per-repo spec template** (Appendix A) and **feature-record template** (`<slug>-feature.md`: goal, target repos, per-repo scope, contract matrix with producer / consumer / contract / shape, shared decisions, integration plan). |
| `references/readme-lifecycle-amendment.md` | Spec line: per-repo `<slug>-design.md`, plus `<slug>-feature.md` in the primary repo for multi-repo features. |

Executor — `plugins/delivery-executor/skills/executing-delivery-handoff/`:

| File | Change |
|---|---|
| `SKILL.md` | Step 1 field list drops scope; Step 2.2 "the spec is this repo's; all of it is yours"; phase 7 per D11; hard rule: never read or follow paths into another repo, even if a spec mentions one — report it as a blocker. |
| `references/handoff-format.md` | Shape and field table per D7 (row `Your scope in this repo` removed; `Spec` row: "this repo's spec; everything in it is yours"). |
| `references/phase-map.md` | Row 7 per D11. |

Unchanged on purpose: both `report-format.md` (Report block stays byte-identical to v0.1.1).

Root and manifests:

- `README.md` — lifecycle table row 0a (per-repo specs + feature record + gate) and the `design_docs/`
  convention line (per-repo spec; feature record in the primary repo).
- `version: "0.1.2"` in both `plugin.json` and three places in `.claude-plugin/marketplace.json`.

### 3.2 Docs (phase 6 will author; listed for completeness)

- `design_docs/architecture/references/delivery-protocol.md` — HandOff contract without `Your scope`;
  spec convention; gate "specs consistent".
- `design_docs/architecture/architecture.md` — new **Contracts & integrations** section (D14), drafted:
  Provides — the `delivery-orchestrator` plugin to Claude Cowork, the `delivery-executor` plugin to Claude
  Code, the HandOff/Report block shapes to both; Consumes — `superpowers` skills (by name), the Claude
  plugin/marketplace system (`marketplace.json`, `plugin.json`, `skills/` discovery).
- ADRs: repo-local specs and docs (D1–D5, D9); gate "specs consistent" (D6); working tree as the authoring
  surface (D10).

### 3.3 Out of scope

- Feature 3 (`delivery-branch-and-pause`) and its spec; the version renumbering of it (D13).
- Any change to the Report block, the stage folders, or phase numbering.
- Shortening the Report (backlog).

## 4. Contracts

**Provides**

| Counterparty | Kind | Shape | Versioning |
|---|---|---|---|
| Claude Cowork (user installs from this marketplace) | plugin `delivery-orchestrator` | skill `orchestrating-delivery`: produces per-repo specs, `<slug>-feature.md`, HandOff blocks, architecture prose | `plugin.json` `version`, tag `vX.Y.Z` |
| Claude Code | plugin `delivery-executor` | skill `executing-delivery-handoff`: consumes a HandOff, prints a Report | same |
| Both sides | HandOff block | line 1 `/delivery-executor:executing-delivery-handoff`; then `Delivery HandOff — <slug> — repo: <repo>`, `Spec`, `Phase to run`, `Gate already passed`, `Do:`, optional `Fix:`, `Acceptance criteria for this repo`, `Report back:` (this release removes `Your scope in this repo`) | both copies change in the same release (parity rule) |
| Both sides | Report block | unchanged from v0.1.1 | byte-identical in both plugins |

**Consumes**

| Counterparty | Kind | Shape | Notes |
|---|---|---|---|
| `superpowers` plugin | skills by name | `brainstorming`, `writing-plans`, `executing-plans` | documented prerequisite, not declared |
| Claude plugin system | manifests + discovery | `.claude-plugin/marketplace.json`, `plugins/<name>/.claude-plugin/plugin.json`, `skills/<skill>/SKILL.md` | `claude plugin validate` must pass |

## 5. Acceptance criteria (gates 3 / 5)

1. `grep -rn "Your scope in this repo" plugins/ README.md` → nothing.
2. Parity: both HandOff blocks open with the slash line and carry the same fields in the same order (`Fix:`
   the existing exception); both Report blocks byte-identical to v0.1.1.
3. D1–D12 each traceable to a concrete sentence in the files of §3.1 (the plan lists file + section per
   decision; gate 3 checks the list).
4. `architecture-doc-template.md` contains the Contracts & integrations section with both tables and the
   D8 columns, the per-repo spec template, and the feature-record template.
5. `documentation-pass.md` done-test contains "without opening any other repository"; Step 0 names the
   working tree and the existing-draft check; Step 5 names the cross-repo-link and ADR-number checks.
6. `claude plugin validate .` and both plugin paths pass; all five `version` occurrences read `0.1.2`.
7. `grep -ri "chatrevenue\|sasha" plugins/` → nothing.
8. Remote install round trip at 0.1.2 passes (phase 4); tag `v0.1.2` on the verified commit.
9. The plan for `repo-local-docs` sits in the stage folder matching the reported phase.

## 6. Integration verification (phase 5, Cowork, read-only)

- At tag `v0.1.2`: five versions read `0.1.2`; parity as in §5.2; `Your scope in this repo` absent.
- Dry run on paper: Cowork re-reads the updated orchestrator skill and walks a hypothetical two-repo
  feature through 0a → "specs consistent" → 0b HandOffs, confirming each step names only per-repo files.
- Phase 6 of this feature is itself the live test of D8–D10 on this repo.

## Appendix A — per-repo spec template (the shape D2 requires; this file follows it)

```
# <Title> — v<X.Y.Z>
- Slug / Status / Author / Release
## 1. Problem            — as it shows up in this repo
## 2. Decisions          — table: decision · alternative rejected · why
## 3. Changes in this repo  — incl. "Out of scope"
## 4. Contracts             — Provides / Consumes, shapes written out
## 5. Acceptance criteria
## 6. Integration verification — only in a single-repo spec; multi-repo: in <slug>-feature.md
```
