# DigiTeam plugin marketplace bootstrap — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn this folder into the public Claude plugin marketplace `digiteam`, shipping two plugins (`delivery-orchestrator` for Cowork, `delivery-executor` for Claude Code) whose skill content Cowork already authored, and publish it as a public GitHub repo tagged `v0.1.0`.

**Architecture:** No application code. The deliverable is a repository shape: one marketplace manifest at `.claude-plugin/marketplace.json`, two plugin folders under `plugins/<name>/` each with `.claude-plugin/plugin.json` + `README.md` + an already-authored `skills/<skill>/` tree, plus root `README.md`, `LICENSE` (MIT), and the delivery-process folders (`design_docs/`, `engineering_plans/{drafts,ongoing,done,documented}/`). "Tests" in this plan are the manifest validator, greps, and a real local marketplace-install round trip — each task runs its check *before* the change (expect fail) and *after* (expect pass).

**Tech Stack:** JSON manifests (Claude plugin/marketplace schema), Markdown, `claude` CLI 2.1.118 (`claude plugin validate|marketplace add|install|list`), `git` 2.46, `gh` 2.76.2 (authenticated as `oleksandr-ieremchuk`, ssh git protocol).

**Spec:** `design_docs/marketplace-bootstrap-design.md`

## Global Constraints

- Marketplace `name` is exactly `digiteam` (spec D5) — it is the suffix users type: `delivery-executor@digiteam`.
- Both plugins are at `version` `0.1.0`; the plugin `name` in `plugin.json` must equal the `name` in its marketplace entry.
- Plugin descriptions in `plugin.json` use the **same text** as the matching marketplace entry (spec §3.3).
- `repository`: `https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace`; `license`: `MIT`.
- No `dependencies` field in either `plugin.json` (spec D4) — the `superpowers` requirement is documented in the READMEs only.
- No `skills` field in either `plugin.json` unless the validator demands one — skills are auto-discovered from `skills/` (spec §3.3).
- Skill text stays generic: `grep -ri "chatrevenue\|sasha" plugins/` must return nothing (spec §7.3, D3).
- Out of scope (spec §6): hooks, agents, MCP servers, CI schema validation, publishing to any curated directory, removing Sasha's personal copy of the skill.
- The 14 files listed under "Cowork-authored content" below are **committed as-is (typo fixes reported as deviations)**.
- Default branch is `main`. `init.defaultBranch` is unset globally, so always pass `-b main`.
- **Line endings stay LF.** Git for Windows had `core.autocrlf=true` in system config, which would have rewritten the commit-as-is files to CRLF on checkout and broken the §7.1 byte-identity claim. Phase 0b set `core.autocrlf=false` and `core.eol=lf` in this repo's local config; gate 1 additionally requires a **tracked `.gitattributes`** with `* text=auto eol=lf` so every clone gets LF regardless of local config (spec §3.1 amended — Task 3 creates it). Verify with `git config core.autocrlf` → `false` before staging anything, and re-verify with `sha256sum -c` after each commit.
- Phase discipline: Tasks 1–6 are **phase 2** (build + validate locally). Tasks 7–8 are **phase 4** (create the GitHub repo, push, tag) and must not start before Cowork's phase-3 deploy-approval gate.

---

## File structure (target state, spec §3.1)

| Path | Responsibility | Produced by |
|---|---|---|
| `.claude-plugin/marketplace.json` | Marketplace manifest: name `digiteam`, owner, the two plugin entries | Task 2 |
| `README.md` | Root docs: what this is, connect-to-Cowork, connect-to-Claude-Code, superpowers prerequisite, phase table, repo conventions, versioning, license | Task 4 |
| `LICENSE` | MIT, © 2026 Oleksandr Ieremchuk | Task 3 |
| `.gitattributes` | `* text=auto eol=lf` — keeps the commit-as-is skill files LF on every clone (spec §3.1, amended at gate 1) | Task 3 |
| `design_docs/marketplace-bootstrap-design.md` | The spec | Cowork — committed in phase 0b |
| `engineering_plans/{drafts,ongoing,done,documented}/.gitkeep` | Plan lifecycle stage folders | phase 0b |
| `engineering_plans/<stage>/marketplace-bootstrap-plan.md` | This plan — written in `drafts/` (phase 0b), moved to `ongoing/` at the start of phase 2, to `done/` at the end of phase 4 | phase 0b |
| `plugins/delivery-orchestrator/.claude-plugin/plugin.json` | Orchestrator plugin manifest | Task 2 |
| `plugins/delivery-orchestrator/README.md` | Which tool it is for, its one skill + trigger phrases, pointer to root README | Task 5 |
| `plugins/delivery-orchestrator/skills/orchestrating-delivery/**` | 9 Cowork-authored files (`SKILL.md` + 8 references) | commit as-is, Task 1 |
| `plugins/delivery-executor/.claude-plugin/plugin.json` | Executor plugin manifest | Task 2 |
| `plugins/delivery-executor/README.md` | Which tool it is for, its one skill + trigger phrases, pointer to root README | Task 5 |
| `plugins/delivery-executor/skills/executing-delivery-handoff/**` | 4 Cowork-authored files (`SKILL.md` + 3 references) | commit as-is, Task 1 |

## Cowork-authored content — commit as-is (typo fixes reported as deviations)

Spec §5 and §7.1 originally said "12 files"; the folder holds **13 files under `plugins/`** (2 × `SKILL.md` + 11 `references/*.md`) plus the spec itself = **14 Cowork-authored files**. Accepted at gate 1 and the spec was corrected to 14, so this is no longer a deviation. The list below is the authoritative inventory.

SHA-256 as re-baselined at the start of phase 2 (the phase-3 review can re-run `sha256sum` against these to prove byte-identity). Rows 1 and 9 are the two files Cowork amended after gate 1; the other 12 hashes are unchanged from planning time, verified by diff before Task 1:

| # | File | sha256 |
|---|---|---|
| 1 | `design_docs/marketplace-bootstrap-design.md` | `4f16dfbd92b7741137b8fa891bf4863d60c94e99a92bdaeef1ec78d4bd844bbf` (re-baselined 2026-09-08 — Cowork amended §3.1, §3.3, §4, §5, §7.1 and the status line after plan review; the phase-0b commit holds the pre-amendment version, Task 1 commits this one) |
| 2 | `plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md` | `e8d1137b0b2400bb50474ae1283a9ec4c552deb516f9ea6c844f2cdccc2bc83e` |
| 3 | `…/orchestrating-delivery/references/architecture-doc-template.md` | `08a3369f53506fa4c9ae3cc10a1e06655d5c4202e1c4861ce7240af98a77dc84` |
| 4 | `…/orchestrating-delivery/references/documentation-pass.md` | `4b7acc70361731f3e80874f555f524f850401e0462b71116cbad5d6eaa8fc1d7` |
| 5 | `…/orchestrating-delivery/references/handoff-prompt.md` | `c034f9739f50eca2b99d24e29b60445e663e613c96efee3ad60d87a4772c9a81` |
| 6 | `…/orchestrating-delivery/references/integration-verification.md` | `7ac64262417da22b95541165474a23b73f58633bef77390548d15f6d0ee4d995` |
| 7 | `…/orchestrating-delivery/references/phase-map.md` | `faf9b8d21d20635a3bfb9014a5d697974bda15cf0e57334e6c605d79176259d1` |
| 8 | `…/orchestrating-delivery/references/readme-lifecycle-amendment.md` | `d301a48ebda4a0fb77e205838241a949cd3a66b824b416342b871c8a4bbf30ba` |
| 9 | `…/orchestrating-delivery/references/report-format.md` | `8f43623f553fe1384a1266a2f335f32bcb7e823c23869bccac486256df6fa2ae` (re-baselined 2026-09-08 — Cowork added the `none — blocked` / `none` variants, closing DP4) |
| 10 | `…/orchestrating-delivery/references/state-discovery.md` | `6fcb3a9352b0a1262ca910c4be2ace5bdc5a3962a1a379fcba39b25f07c31d79` |
| 11 | `plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md` | `85e631d617a352de4a6333e959bab25316aec9cbf9a5f959817943b1562425b0` |
| 12 | `…/executing-delivery-handoff/references/handoff-format.md` | `899a4e9fe09179891bb6e2c563b515454db79d6a5b62accc3350665f06dbcf20` |
| 13 | `…/executing-delivery-handoff/references/phase-map.md` | `fbe171fe54aeb7b3e867f02dfebe30006c93f5af76fe78e9bb52fe5b1031f020` |
| 14 | `…/executing-delivery-handoff/references/report-format.md` | `9927f86548bb8af9da820233ad5267082f42fed129ec6e46ab0159edc21cb45f` |

Already checked read-only during planning: the §5 truncation caveat looks resolved — `SKILL.md`, `references/phase-map.md` and `references/state-discovery.md` all end on complete sentences; `grep -ri "chatrevenue\|sasha" plugins/` already returns nothing; every `references/*.md` named in either `SKILL.md` exists at that path. Sasha's own diff against his original copy stays a pre-tag step (spec §5) and is outside this plan.

## Decisions this plan takes (flag at the phase-1 gate)

- **DP1 — `plugin.json` `author` carries no email. Accepted at gate 1 and folded into the spec (§3.3 now says `author: { "name": "Oleksandr Ieremchuk" }`), so it is no longer a deviation.** The original conflict: §3.3 said `author` = the owner (`oleksandr.ieremchuk@chatrevenue.ai`) while §7.3 requires `grep -ri "chatrevenue" plugins/` to return nothing. The email stays in root `marketplace.json`, outside `plugins/`.
- **DP2 — `marketplace.json` shape falls back if the validator objects. Outcome (phase 2): only `$schema` was dropped.** The manifest was written per §3.2 (`metadata.description` + `metadata.version`) plus a `$schema` line the plan added on its own; `claude plugin validate .` rejected exactly one key — `root: Unrecognized key: "$schema"` — so it was removed and all three validations then passed. Everything the spec's §3.2 JSON actually specifies survives: `metadata.description`, `metadata.version`, per-plugin `version`, `category` and `keywords` are all accepted. No `name`, `source` or `description` was touched.
- **DP3 — the tag is plain `v0.1.0`.** Spec §3.5 asks for `v0.1.0`. Note that `claude plugin tag` produces `{name}--v{version}` tags instead, so it is not used. Both plugins share one version, so a single repo-level tag is unambiguous.
- **DP4 — protocol delta, resolved by Cowork before phase 2, nothing for Code to do.** The executor's `report-format.md` allowed two enum values the orchestrator's copy omitted (`Phase completed: none — blocked`, `Plan stage now: none`). Cowork amended the orchestrator's `report-format.md` (re-baselined, row 9 above) and §4 of the spec; the two Report blocks are now byte-identical, verified with `diff <(awk '/^Delivery Report/,/^Ready for gate/' …orchestrator…) <(awk … …executor…)` → no output.

---

## PHASE 2 — build + validate locally (Tasks 1–6)

### Task 1: Commit the Cowork-authored plugin content as-is

**Files:**
- Commit (no edits): the 13 files under `plugins/` from the inventory above, **plus the amended `design_docs/marketplace-bootstrap-design.md`** — phase 0b committed its pre-amendment form, so it shows as `M` and travels in this commit as content, not as a rewrite by Code
- Test: the reference-integrity loop and the generic-wording grep, plus `sha256sum -c`

**Interfaces:**
- Consumes: the working folder as Cowork left it (phase 0b already committed `design_docs/`, `engineering_plans/` and this plan)
- Produces: `plugins/delivery-orchestrator/skills/orchestrating-delivery/` and `plugins/delivery-executor/skills/executing-delivery-handoff/` tracked in git at the hashes above — the `source` targets Task 2's manifests point at

- [x] **Step 1: Record the pre-commit hashes as the byte-identity baseline**

```bash
find plugins design_docs -type f | sort | xargs sha256sum > /tmp/cowork-authored.sha256
cat /tmp/cowork-authored.sha256
```

Expected: 14 lines matching the inventory table exactly. If any hash differs, stop — the folder changed since planning. Report it as a blocker rather than committing silently.

- [x] **Step 2: Run the reference-integrity check (spec §7.4) before committing**

Every `references/<file>.md` named in a `SKILL.md` must exist next to it:

```bash
for skill in plugins/*/skills/*/SKILL.md; do
  dir=$(dirname "$skill")
  grep -o 'references/[a-z-]*\.md' "$skill" | sort -u | while read -r ref; do
    test -f "$dir/$ref" && echo "OK   $dir/$ref" || echo "MISS $dir/$ref"
  done
done
```

Expected: only `OK` lines — 8 for the orchestrator, 3 for the executor. Any `MISS` is a blocker (a commit-as-is file references something Cowork did not deliver); report it, do not invent the missing file.

- [x] **Step 3: Run the generic-wording check (spec §7.3)**

```bash
grep -rin "chatrevenue\|sasha" plugins/ ; echo "exit=$?"
```

Expected: no output and `exit=1`. A match is a blocker — do not rewrite skill text to make it pass; report it.

- [x] **Step 4: Confirm nothing else is staged, then stage `plugins/` and the amended spec**

```bash
git status --short
git add plugins/ design_docs/marketplace-bootstrap-design.md
git status --short
```

Expected: before, `?? plugins/` plus `M design_docs/marketplace-bootstrap-design.md`; after, 13 `A` lines under `plugins/` and one `M` for the spec, nothing else.

- [x] **Step 5: Verify the staged bytes equal the on-disk bytes**

```bash
git diff --cached --stat | tail -1
sha256sum -c /tmp/cowork-authored.sha256
```

Expected: `14 files changed, …`; every `sha256sum -c` line reads `OK`.

- [x] **Step 6: Commit**

```bash
git commit -m "$(printf '%s\n' 'feat: add Cowork-authored delivery skills as-is' '' 'Two skill trees, byte-identical to what Cowork authored: orchestrating-delivery' '(SKILL.md + 8 references) and executing-delivery-handoff (SKILL.md + 3' 'references). Also carries Cowork post-gate-1 amendments to the spec and to the' "orchestrator's report-format.md (the blocked-variant parity fix)." '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

- [x] **Step 7: Verify the commit holds exactly the 14 files**

```bash
git show --stat --name-only HEAD | grep -c '^plugins/'
git show --name-only --format="" HEAD | grep -c 'design_docs/'
```

Expected: `13`, then `1`.

---

### Task 2: The three manifests (marketplace + both plugins)

**Files:**
- Create: `.claude-plugin/marketplace.json`
- Create: `plugins/delivery-orchestrator/.claude-plugin/plugin.json`
- Create: `plugins/delivery-executor/.claude-plugin/plugin.json`
- Test: `claude plugin validate .`, `claude plugin validate ./plugins/delivery-orchestrator`, `claude plugin validate ./plugins/delivery-executor`, plus the contract script in Step 6

**Interfaces:**
- Consumes: the skill trees committed in Task 1 — each `source` must resolve to a folder holding `plugin.json` + `skills/*/SKILL.md`
- Produces: marketplace name `digiteam`, plugin names `delivery-orchestrator` and `delivery-executor` — the exact strings Tasks 4–6 document and install (`delivery-executor@digiteam`)

- [x] **Step 1: Run the validator first, to see it fail**

```bash
claude plugin validate .
```

Expected: FAIL — there is no `.claude-plugin/marketplace.json` here yet. Record the exact wording; it is the "before" half of the §7.2 evidence.

- [x] **Step 2: Write `.claude-plugin/marketplace.json`**

```bash
mkdir -p .claude-plugin
cat > .claude-plugin/marketplace.json <<'EOF'
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "digiteam",
  "owner": {
    "name": "Oleksandr Ieremchuk",
    "email": "oleksandr.ieremchuk@chatrevenue.ai"
  },
  "metadata": {
    "description": "DigiTeam plugins for Claude Cowork and Claude Code — multi-repo feature delivery orchestration.",
    "version": "0.1.0"
  },
  "plugins": [
    {
      "name": "delivery-orchestrator",
      "source": "./plugins/delivery-orchestrator",
      "description": "Cowork side of the delivery process: locate a feature's phase across repos, run the review gates, hand work to Claude Code, verify integration, author docs.",
      "version": "0.1.0",
      "category": "productivity",
      "keywords": ["delivery", "orchestration", "cowork", "multi-repo", "handoff"]
    },
    {
      "name": "delivery-executor",
      "source": "./plugins/delivery-executor",
      "description": "Claude Code side of the delivery process: execute a Delivery HandOff in one repo (plan, implement, deploy+verify, review docs, move plan stages) and print a Delivery Report.",
      "version": "0.1.0",
      "category": "productivity",
      "keywords": ["delivery", "handoff", "report", "claude-code", "plan-stages"]
    }
  ]
}
EOF
python -c "import json;json.load(open('.claude-plugin/marketplace.json'));print('valid JSON')"
```

Expected: `valid JSON`.

- [x] **Step 3: Write `plugins/delivery-orchestrator/.claude-plugin/plugin.json`**

Per DP1 the `author` carries a name only; no `dependencies`, no `skills`.

```bash
mkdir -p plugins/delivery-orchestrator/.claude-plugin
cat > plugins/delivery-orchestrator/.claude-plugin/plugin.json <<'EOF'
{
  "name": "delivery-orchestrator",
  "version": "0.1.0",
  "description": "Cowork side of the delivery process: locate a feature's phase across repos, run the review gates, hand work to Claude Code, verify integration, author docs.",
  "author": { "name": "Oleksandr Ieremchuk" },
  "repository": "https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace",
  "license": "MIT",
  "keywords": ["delivery", "orchestration", "cowork", "multi-repo", "handoff"]
}
EOF
python -c "import json;json.load(open('plugins/delivery-orchestrator/.claude-plugin/plugin.json'));print('valid JSON')"
```

Expected: `valid JSON`.

- [x] **Step 4: Write `plugins/delivery-executor/.claude-plugin/plugin.json`**

```bash
mkdir -p plugins/delivery-executor/.claude-plugin
cat > plugins/delivery-executor/.claude-plugin/plugin.json <<'EOF'
{
  "name": "delivery-executor",
  "version": "0.1.0",
  "description": "Claude Code side of the delivery process: execute a Delivery HandOff in one repo (plan, implement, deploy+verify, review docs, move plan stages) and print a Delivery Report.",
  "author": { "name": "Oleksandr Ieremchuk" },
  "repository": "https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace",
  "license": "MIT",
  "keywords": ["delivery", "handoff", "report", "claude-code", "plan-stages"]
}
EOF
python -c "import json;json.load(open('plugins/delivery-executor/.claude-plugin/plugin.json'));print('valid JSON')"
```

Expected: `valid JSON`.

- [x] **Step 5: Run all three validations to verify they pass**

```bash
claude plugin validate .
claude plugin validate ./plugins/delivery-orchestrator
claude plugin validate ./plugins/delivery-executor
```

Expected: PASS with no errors, three times. Paste the actual output into the Report — this is the §7.2 evidence.

If the marketplace validation rejects `metadata` (or `$schema`, or the per-plugin `version`/`keywords`), apply DP2: drop the rejected field, move the description to a top-level `description` if `metadata` goes, re-run until clean, and record each dropped field as a deviation. If a plugin validation demands an explicit skills declaration, add `"skills": ["./skills"]` and record that as a deviation too (spec §3.3 permits it only under validator pressure).

- [x] **Step 6: Confirm the name/source contract mechanically**

This is also the check Cowork re-runs at the phase-5 integration gate (spec §8):

```bash
python - <<'EOF'
import json, os
m = json.load(open('.claude-plugin/marketplace.json'))
assert m['name'] == 'digiteam', m['name']
for entry in m['plugins']:
    src = entry['source']
    pj = json.load(open(os.path.join(src, '.claude-plugin', 'plugin.json')))
    assert pj['name'] == entry['name'], (pj['name'], entry['name'])
    assert pj['description'] == entry['description'], entry['name']
    assert pj['version'] == '0.1.0', pj['version']
    assert 'dependencies' not in pj, entry['name']
    skills = os.listdir(os.path.join(src, 'skills'))
    assert skills, src
    for s in skills:
        assert os.path.isfile(os.path.join(src, 'skills', s, 'SKILL.md')), s
    print('OK', entry['name'], '->', src, 'skills:', skills)
print('contract OK')
EOF
```

Expected: two `OK` lines, then `contract OK`.

- [x] **Step 7: Commit**

```bash
git add .claude-plugin plugins/delivery-orchestrator/.claude-plugin plugins/delivery-executor/.claude-plugin
git status --short
git commit -m "$(printf '%s\n' 'feat: add digiteam marketplace and both plugin manifests' '' 'marketplace.json declares the digiteam marketplace with delivery-orchestrator' 'and delivery-executor at 0.1.0; each plugin carries its own manifest. Skills' 'are auto-discovered from skills/; no dependencies field (superpowers is a' 'documented prerequisite instead).' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

Expected: exactly 3 files added.

---

### Task 3: LICENSE (MIT) and `.gitattributes`

**Files:**
- Create: `LICENSE`
- Create: `.gitattributes`
- Test: `head -3 LICENSE`; `git check-attr text eol -- <a skill file>`; `sha256sum -c` still clean after the attributes file exists

**Interfaces:**
- Consumes: nothing
- Produces: the file both `plugin.json` `"license": "MIT"` fields refer to and the root README's License section links to (Task 4), plus the repo-level LF guarantee the §7.1 byte-identity criterion rests on

- [x] **Step 1: Verify it is missing**

```bash
test -f LICENSE && echo present || echo absent
```

Expected: `absent`.

- [x] **Step 2: Write the MIT license**

```bash
cat > LICENSE <<'EOF'
MIT License

Copyright (c) 2026 Oleksandr Ieremchuk

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF
head -3 LICENSE
```

Expected: `MIT License`, a blank line, then the copyright line.

- [x] **Step 3: Write `.gitattributes`** (spec §3.1, amended at gate 1)

```bash
printf '%s\n' '* text=auto eol=lf' > .gitattributes
cat .gitattributes
git check-attr text eol -- plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md
```

Expected: the file reads `* text=auto eol=lf`, and `check-attr` reports `text: auto` and `eol: lf` for a skill file.

- [x] **Step 4: Verify the attributes file changed no bytes**

```bash
git status --short
sha256sum -c /tmp/cowork-authored.sha256
```

Expected: only `?? .gitattributes` plus whatever else is legitimately new — **no `M` line for any committed skill file** (the blobs are already LF, so renormalisation is a no-op). Every `sha256sum -c` line still `OK`. An `M` on a skill file means git wants to rewrite the content: stop and report, do not commit it.

- [x] **Step 5: Commit**

```bash
git add LICENSE .gitattributes
git commit -m "$(printf '%s\n' 'chore: add MIT license and LF line-ending policy' '' 'text=auto eol=lf keeps the commit-as-is skill files LF on every clone, which is' 'what the byte-identity acceptance criterion rests on.' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

---

### Task 4: Root README

**Files:**
- Create: `README.md`
- Read for content: `plugins/delivery-orchestrator/skills/orchestrating-delivery/references/phase-map.md` (the phase table is copied from it)
- Test: the section/command grep sweep in Step 3, the phase-table diff in Step 4, the JSON-block parse in Step 5

**Interfaces:**
- Consumes: the marketplace name `digiteam` and both plugin names from Task 2
- Produces: the canonical install instructions the per-plugin READMEs (Task 5) link back to, and the §7.6 artifact ("README renders with the connect instructions for both tools")

- [x] **Step 1: Verify it is missing**

```bash
test -f README.md && echo present || echo absent
```

Expected: `absent`.

- [x] **Step 2: Write the README with the eight sections of spec §3.4, in order**

Write the file with the Write tool (its content contains nested fenced code blocks, which a heredoc inside a fenced block cannot carry safely). Content, verbatim:

````markdown
# DigiTeam plugin marketplace

Two Claude plugins, one delivery process. Cowork orchestrates a feature across
repositories and decides the gates; Claude Code does everything that touches
code and git inside a single repo. The two sides talk only through two
copy-paste chat blocks — a **HandOff** (Cowork → Code) and a **Report**
(Code → Cowork).

```
Cowork                                  Claude Code
──────                                  ───────────
delivery-orchestrator                   delivery-executor
 spec, gates, integration verify
        │
        │  Delivery HandOff (chat) ────────►  run exactly one phase in one repo:
        │                                     plan / execute / deploy+verify /
        │                                     review docs / move plan stage
        ◄──────── Delivery Report (chat) ─────┘
 decide the gate: pass → next phase
                 fail → corrective HandOff
```

| Plugin | Install into | Skill | What it does |
|---|---|---|---|
| `delivery-orchestrator` | Cowork | `orchestrating-delivery` | Locates a feature's phase across all its repos, runs the review gates, emits HandOffs, runs cross-repo integration verification, authors architecture docs. Never touches git. |
| `delivery-executor` | Claude Code | `executing-delivery-handoff` | Consumes a pasted HandOff, verifies repo + plan stage against the requested phase, runs exactly that phase, keeps the plan-stage folders truthful with `git mv`, prints the Report. |

## Connect to Cowork

1. **Customize → Plugins → Add marketplace**
2. Paste `https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace`
3. Install **`delivery-orchestrator`**.

The `orchestrating-delivery` skill then triggers on phrases like "where are we
on this feature", "hand this off to Code", "the plan is ready", "approve
deploy", "run integration verification".

Cowork caches plugin contents: after a new version is pushed here, uninstall and
reinstall the plugin (or remove and re-add the marketplace) to pick it up.

## Connect to Claude Code

```
/plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace
/plugin install delivery-executor@digiteam
```

Or from a shell:

```bash
claude plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace
claude plugin install delivery-executor@digiteam
```

For a whole team, commit this to the repo's `.claude/settings.json` instead, so
every checkout gets the plugin:

```json
{
  "extraKnownMarketplaces": {
    "digiteam": {
      "source": {
        "source": "github",
        "repo": "oleksandr-ieremchuk/digiteam-cowork-marketplace"
      }
    }
  },
  "enabledPlugins": {
    "delivery-executor@digiteam": true
  }
}
```

## Prerequisite: superpowers

Both skills delegate the thinking-heavy steps to the `superpowers` skills —
`superpowers:brainstorming` (phase 0a), `superpowers:writing-plans` (0b) and
`superpowers:executing-plans` (2). Install that marketplace too:

```bash
claude plugin marketplace add obra/superpowers-marketplace
claude plugin install superpowers@superpowers-marketplace
```

Marketplace: <https://github.com/obra/superpowers-marketplace>

This is a documented prerequisite rather than a declared `dependencies` entry:
superpowers lives in a third-party marketplace, so a hard dependency would need
an allowlist and would break on a rename.

## The process in one screen

| # | Phase | Owner | Tool / artifact | Gate |
|---|-------|-------|-----------------|------|
| 0a | Brainstorm + spec (across all target repos) | Cowork | `superpowers:brainstorming` → spec in `design_docs/` (lists target repos + per-repo scope) | — |
| 0b | Plan from spec | Code (per repo) | HandOff → Code runs `superpowers:writing-plans` → plan in `drafts/` | — |
| 1 | Plan review | Cowork | read each draft plan; check it against the spec | **plan approved** |
| 2 | Execution | Code (per repo) | plan `drafts → ongoing`; `superpowers:executing-plans` | — |
| 3 | Change review + deploy approval | Cowork | read Code's Report + resulting files (read-only) against the plan | **deploy approved** |
| 4 | Deploy + functional verification | Code | deploy, verify in-repo, plan `ongoing → done`, print a Report | — |
| 5 | Acceptance + integration verification | Cowork | cross-repo, read-only, end-to-end (the join point) | **acceptance approved** |
| 6 | Documentation | Cowork | the documentation pass, prose only, no commit | — |
| 7 | Documentation review | Code | confirm the docs match the real code | **docs approved** |
| 8 | Plan `done → documented` | Code | git move + commit | — |

A failed Cowork gate (1, 3, 5) does not advance the feature: Cowork issues a
corrective HandOff, Code answers with a new Report, and the same gate is
re-checked. Looping there is the normal path, not an error. In a multi-repo
feature, phases 0b–4 and 7–8 run per repo in parallel; phase 5 is the join and
starts only once **every** target repo reports `done`.

## Repo conventions

The process expects two things in every repo it drives, and this repo follows
them itself:

- `design_docs/` — one spec per feature, `<slug>-design.md`, listing the target
  repos and the per-repo scope.
- `engineering_plans/{drafts,ongoing,done,documented}/` — one plan per feature
  per repo. **The stage folder is the durable phase signal:** `drafts` planned,
  `ongoing` executing, `done` shipped + functionally verified, `documented`
  folded into the architecture docs. Only Claude Code moves a plan between
  stages, always as a `git mv` plus a commit.

## Versioning

Bump `version` in `.claude-plugin/marketplace.json` (both the `metadata` block
and the affected plugin entries) and in each changed
`plugins/*/.claude-plugin/plugin.json`, keeping the two in agreement; then tag
the repo `vX.Y.Z` and push the tag.

## License

MIT — see [LICENSE](LICENSE).
````

- [x] **Step 3: Verify every required section and command is present**

```bash
for s in "^# DigiTeam plugin marketplace" "^## Connect to Cowork" "^## Connect to Claude Code" \
         "^## Prerequisite: superpowers" "^## The process in one screen" "^## Repo conventions" \
         "^## Versioning" "^## License"; do
  grep -q "$s" README.md && echo "OK   $s" || echo "MISS $s"
done
grep -q "plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace" README.md && echo "OK   code install"
grep -q "plugin install delivery-executor@digiteam" README.md && echo "OK   plugin install"
grep -q "extraKnownMarketplaces" README.md && grep -q "enabledPlugins" README.md && echo "OK   team settings"
grep -q "https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace" README.md && echo "OK   cowork url"
grep -q "superpowers-marketplace" README.md && echo "OK   superpowers link"
```

Expected: eight `OK` section lines plus five more `OK` lines, and no `MISS`.

- [x] **Step 4: Verify the phase table did not drift from the skill's own phase map**

```bash
diff <(grep '^| [0-9]' plugins/delivery-orchestrator/skills/orchestrating-delivery/references/phase-map.md) \
     <(grep '^| [0-9]' README.md)
```

Expected: at most the shortened phase-6 tool cell (the README drops the `references/documentation-pass.md` path, which means nothing outside the plugin). Any different owner or gate wording is a defect — fix the README, never the skill file.

- [x] **Step 5: Verify the fenced JSON block parses**

```bash
python - <<'EOF'
import json, re, io
blocks = re.findall(r"```json\n(.*?)```", open('README.md', encoding='utf-8').read(), re.S)
for b in blocks:
    json.load(io.StringIO(b))
print(len(blocks), 'json block(s) parse')
EOF
```

Expected: `1 json block(s) parse`.

- [x] **Step 6: Commit**

```bash
git add README.md
git commit -m "$(printf '%s\n' 'docs: add root README with connect instructions for both tools' '' 'Covers what the marketplace is, the Cowork and Claude Code install paths' '(including the team .claude/settings.json variant), the superpowers' 'prerequisite, the phase table, repo conventions, versioning and license.' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

---

### Task 5: Per-plugin READMEs

**Files:**
- Create: `plugins/delivery-orchestrator/README.md`
- Create: `plugins/delivery-executor/README.md`
- Test: the grep checks in Step 4 and a re-validation of both plugins in Step 5

**Interfaces:**
- Consumes: the root README from Task 4 — both link back to `../../README.md`
- Produces: nothing later tasks depend on, except that Task 6's §7.3 grep now also covers these two files

- [x] **Step 1: Verify both are missing**

```bash
ls plugins/*/README.md 2>/dev/null || echo "none yet"
```

Expected: `none yet`.

- [x] **Step 2: Write the orchestrator README**

```bash
cat > plugins/delivery-orchestrator/README.md <<'EOF'
# delivery-orchestrator

**Install this into Claude Cowork.** It is the orchestrating half of the DigiTeam
delivery process: it reasons about a feature across *all* of its repositories,
decides the review gates, and hands the code work to Claude Code, where the
companion plugin `delivery-executor` runs it.

It contains one skill, **`orchestrating-delivery`**, which triggers on phrases
like "where are we on this feature", "what's next for this feature", "start
delivery", "hand this off to Code", "the plan is ready", "review the changes",
"approve deploy", "run integration verification", "accept this work" — and
their equivalents in other languages.

It owns phases 0a (brainstorm + spec), 1 (plan review), 3 (change review +
deploy approval), 5 (acceptance + cross-repo integration verification) and 6
(the documentation pass). It never runs `git` or `gh`, never commits, and never
moves a plan between stage folders — those belong to Claude Code, every time.

Requires the `superpowers` marketplace (`superpowers:brainstorming`,
`superpowers:writing-plans`, `superpowers:executing-plans`).

Install instructions, the full phase table and the repo conventions are in the
[marketplace README](../../README.md).
EOF
```

- [x] **Step 3: Write the executor README**

```bash
cat > plugins/delivery-executor/README.md <<'EOF'
# delivery-executor

**Install this into Claude Code.** It is the executing half of the DigiTeam
delivery process: it runs one phase, in one repository, from one HandOff pasted
out of Cowork, where the companion plugin `delivery-orchestrator` produced it.

It contains one skill, **`executing-delivery-handoff`**, which triggers on a
pasted `Delivery HandOff — <slug> — repo: <name>` block, and on phrases like
"run this handoff", "here's the handoff", "execute the delivery handoff",
"print the delivery report", "move the plan to ongoing/done/documented" — and
their equivalents in other languages.

It verifies the repo and the plan's stage folder against the requested phase,
runs exactly phase 0b (write the plan), 2 (execute), 4 (deploy + functional
verification), 7 (review docs) or 8 (move `done → documented`), keeps the stage
folders truthful with `git mv` plus a commit, and finishes by printing a
Delivery Report for the user to paste back into Cowork. It never widens scope
beyond the HandOff.

Requires the `superpowers` marketplace (`superpowers:writing-plans`,
`superpowers:executing-plans`).

Install instructions, the full phase table and the repo conventions are in the
[marketplace README](../../README.md).
EOF
```

- [x] **Step 4: Verify both READMEs cover the required points and stay generic**

```bash
grep -q "Cowork" plugins/delivery-orchestrator/README.md \
  && grep -q "orchestrating-delivery" plugins/delivery-orchestrator/README.md \
  && grep -q "../../README.md" plugins/delivery-orchestrator/README.md \
  && echo "OK orchestrator"
grep -q "Claude Code" plugins/delivery-executor/README.md \
  && grep -q "executing-delivery-handoff" plugins/delivery-executor/README.md \
  && grep -q "../../README.md" plugins/delivery-executor/README.md \
  && echo "OK executor"
grep -rin "chatrevenue\|sasha" plugins/ ; echo "generic-grep exit=$?"
```

Expected: `OK orchestrator`, `OK executor`, and `generic-grep exit=1` with no matches.

- [x] **Step 5: Re-validate — the new files must not break either plugin**

```bash
claude plugin validate ./plugins/delivery-orchestrator
claude plugin validate ./plugins/delivery-executor
```

Expected: PASS, twice.

- [x] **Step 6: Commit**

```bash
git add plugins/delivery-orchestrator/README.md plugins/delivery-executor/README.md
git commit -m "$(printf '%s\n' 'docs: add per-plugin READMEs' '' 'Each names the tool it belongs in, the single skill it ships and its trigger' 'phrases, and points back to the marketplace README.' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

---

### Task 6: Local install test and the full §7 acceptance sweep

**Files:**
- Modify: none — this task only reads the repo and mutates local `claude` plugin state, which it cleans up again
- Test: `claude plugin marketplace add ./`, `claude plugin install delivery-executor@digiteam`, `claude plugin list`, then the §7 sweep

**Interfaces:**
- Consumes: everything built in Tasks 1–5
- Produces: the phase-2 evidence block for the Delivery Report — real command output for spec §7 criteria 1–5 and 7

- [x] **Step 1: Record the pre-existing local plugin state, so cleanup can restore it**

```bash
claude plugin marketplace list
claude plugin list
```

Expected: `digiteam` is **not** listed. If it is (a leftover from an earlier run), remove it first with `claude plugin marketplace remove digiteam`.

- [x] **Step 2: Add this repo as a local marketplace (spec §7.5)**

```bash
claude plugin marketplace add ./
claude plugin marketplace list
```

Expected: success, and `digiteam` appears with a local source pointing at this folder. If `./` is rejected, retry with the absolute path and record the substitution as a deviation.

- [x] **Step 3: Install the executor plugin**

```bash
claude plugin install delivery-executor@digiteam
claude plugin list
```

Expected: the install succeeds and `delivery-executor` appears as installed/enabled.

- [x] **Step 4: Confirm the skill is discoverable**

```bash
find ~/.claude/plugins -path '*delivery-executor*' -name 'SKILL.md'
grep -m1 '^name:' plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md
```

Expected: a `SKILL.md` path under the installed plugin, and `name: executing-delivery-handoff`. Note in the Report that the human-facing confirmation is the skill list inside an interactive `claude` session; this session is non-interactive, so `claude plugin list` plus the installed `SKILL.md` path is the evidence recorded.

- [x] **Step 5: Install the orchestrator too, proving both sources resolve**

```bash
claude plugin install delivery-orchestrator@digiteam
claude plugin list
find ~/.claude/plugins -path '*delivery-orchestrator*' -name 'SKILL.md'
```

Expected: the install succeeds and the orchestrator's `SKILL.md` is present under the installed copy.

- [x] **Step 6: Run the whole §7 acceptance sweep in one pass and capture the output**

```bash
echo "== 7.1 layout + authored files =="
find . -path ./.git -prune -o -type f -print | sort
sha256sum -c /tmp/cowork-authored.sha256
echo "== 7.2 manifests valid =="
claude plugin validate . && claude plugin validate ./plugins/delivery-orchestrator && claude plugin validate ./plugins/delivery-executor
echo "== 7.3 no personal references =="
grep -rin "chatrevenue\|sasha" plugins/ ; echo "exit=$?"
echo "== 7.4 every referenced reference exists =="
for skill in plugins/*/skills/*/SKILL.md; do d=$(dirname "$skill"); grep -o 'references/[a-z-]*\.md' "$skill" | sort -u | while read -r r; do test -f "$d/$r" || echo "MISS $d/$r"; done; done; echo "refs checked"
echo "== 7.7 stage folders + plan location =="
ls engineering_plans/*/
ls engineering_plans/ongoing/marketplace-bootstrap-plan.md
echo "== tree clean =="
git status --short; git log --oneline
```

Expected: the §3.1 layout with no extra files; every `sha256sum -c` line `OK`; three validator passes; `exit=1` on the grep with no matches; `refs checked` with no `MISS`; four stage folders with the plan in `ongoing/` (phase 2 moved it there at its start; phase 4 moves it to `done/`); a clean tree.

- [x] **Step 7: Clean up the local install state**

```bash
claude plugin uninstall delivery-executor
claude plugin uninstall delivery-orchestrator
claude plugin marketplace remove digiteam
claude plugin marketplace list
```

Expected: `digiteam` gone and the machine back to its pre-test state. Leaving the local-path install in place would shadow the GitHub-sourced install tested in phase 4.

- [x] **Step 8: No commit — this task changes no files**

```bash
git status --short
```

Expected: empty output. **Phase 2 ends here:** print the Delivery Report and stop. Creating the GitHub repo needs Cowork's phase-3 deploy approval.

---

## PHASE 4 — deploy: publish to GitHub (Tasks 7–8), only after the deploy-approval gate

### Task 7: Create the public GitHub repo and push `main`

**Files:**
- Modify: git remote configuration only — no repo file changes
- Test: `gh repo view --json visibility,defaultBranchRef`, `git diff --stat HEAD origin/main`, a real install from the GitHub source

**Interfaces:**
- Consumes: the validated, fully committed `main` from Tasks 1–5
- Produces: the public URL both READMEs already advertise, and the `origin` remote Task 8 pushes the tag to

- [x] **Step 1: Confirm the gate and the preconditions**

```bash
git status --short
git branch --show-current
git log --oneline
gh auth status
gh repo view oleksandr-ieremchuk/digiteam-cowork-marketplace 2>&1 | head -3
```

Expected: a clean tree; branch `main`; the phase-0b and phase-2 commits; `gh` authenticated as `oleksandr-ieremchuk`; and the repo view **failing** with "Could not resolve to a Repository" — it must not exist yet. If it does exist, stop and report a blocker: do not push into a pre-existing repo without Cowork's say.

- [x] **Step 2: Create the public repo (no push yet)**

```bash
gh repo create oleksandr-ieremchuk/digiteam-cowork-marketplace \
  --public \
  --description "DigiTeam plugin marketplace for Claude Cowork and Claude Code"
gh repo view oleksandr-ieremchuk/digiteam-cowork-marketplace --json name,visibility,description
```

Expected: created, with `"visibility": "PUBLIC"` and exactly that description.

- [x] **Step 3: Add the remote and push `main`**

```bash
git remote add origin git@github.com:oleksandr-ieremchuk/digiteam-cowork-marketplace.git
git remote -v
git push -u origin main
```

Expected: the push succeeds and `main` tracks `origin/main`. (`gh auth status` reports ssh as the git protocol; if ssh fails, switch the remote to `https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace.git` and record the substitution as a deviation.)

- [x] **Step 4: Verify the pushed state (spec §7.6)**

```bash
gh repo view oleksandr-ieremchuk/digiteam-cowork-marketplace --json visibility,defaultBranchRef,url
git log origin/main --oneline
git diff --stat HEAD origin/main
gh api repos/oleksandr-ieremchuk/digiteam-cowork-marketplace/contents/.claude-plugin/marketplace.json --jq '.name'
```

Expected: `PUBLIC`, default branch `main`, the same commits locally and remotely, an empty diff, and the API returning `marketplace.json` — which is exactly the reachability Cowork's phase-5 check needs.

- [x] **Step 5: Verify the remote install path end to end**

These are the same commands the README gives users:

```bash
claude plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace
claude plugin marketplace list
claude plugin install delivery-executor@digiteam
claude plugin list
claude plugin uninstall delivery-executor
claude plugin marketplace remove digiteam
```

Expected: the GitHub-sourced marketplace adds cleanly, the plugin installs from it, and the cleanup leaves no trace.

- [x] **Step 6: Confirm the README is served by GitHub**

```bash
gh api repos/oleksandr-ieremchuk/digiteam-cowork-marketplace/readme --jq '.name, .size'
```

Expected: `README.md` and a non-zero size. Note in the Report that visual rendering (tables, the ASCII diagram) is Sasha's one-click check at the repo URL.

---

### Task 8: Tag `v0.1.0` and move the plan to `done`

**Files:**
- Modify: git tags, then `engineering_plans/ongoing/marketplace-bootstrap-plan.md` → `engineering_plans/done/`
- Test: `git ls-remote --tags origin`, `gh api …/git/refs/tags/v0.1.0`, `ls engineering_plans/done/`

**Interfaces:**
- Consumes: the pushed `main` from Task 7, verified by its Steps 4–6
- Produces: the `v0.1.0` release marker the README's versioning section describes, and the `done` plan stage the phase-5 acceptance gate reads

- [x] **Step 1: Confirm Task 7's verification passed and no tag exists**

```bash
git ls-remote --tags origin
git tag
```

Expected: both empty. Per spec §3.5 the tag goes on **after** the push passes verification — if anything in Task 7 Steps 4–6 failed, stop and report instead of tagging.

- [x] **Step 2: Check the version agreement one last time**

```bash
python - <<'EOF'
import json
m = json.load(open('.claude-plugin/marketplace.json'))
versions = {p['name']: p.get('version') for p in m['plugins']}
versions['marketplace.metadata'] = m.get('metadata', {}).get('version')
for p in m['plugins']:
    pj = json.load(open(p['source'] + '/.claude-plugin/plugin.json'))
    assert pj['version'] == '0.1.0', (p['name'], pj['version'])
print(versions)
EOF
```

Expected: every version present reads `0.1.0`. `marketplace.metadata` is `None` only if DP2 forced the `metadata` block out — already reported as a deviation in that case.

- [x] **Step 3: Create and push the annotated tag**

```bash
git tag -a v0.1.0 -m "digiteam marketplace v0.1.0 — delivery-orchestrator + delivery-executor"
git push origin v0.1.0
```

- [x] **Step 4: Verify the tag landed on the pushed commit**

```bash
git ls-remote --tags origin
gh api repos/oleksandr-ieremchuk/digiteam-cowork-marketplace/git/refs/tags/v0.1.0 --jq '.ref'
git rev-list -n1 v0.1.0
git rev-parse HEAD
```

Expected: `refs/tags/v0.1.0` present remotely, and the tag's target commit equal to local `HEAD`.

- [x] **Step 5: Move the plan `ongoing → done` and commit (phase 4's stage move)**

```bash
git mv engineering_plans/ongoing/marketplace-bootstrap-plan.md engineering_plans/done/marketplace-bootstrap-plan.md
git commit -m "$(printf '%s\n' 'chore: move marketplace-bootstrap plan ongoing -> done' '' 'Deployed: public repo created, main pushed, v0.1.0 tagged and verified.' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
git push
ls engineering_plans/done/
```

Expected: the plan sits in `engineering_plans/done/` both locally and on `origin/main` (spec §7.7). Then print the Report with `Plan stage now: done` and `Ready for gate: acceptance`.

---

## Acceptance-criteria traceability (spec §7)

| §7 criterion | Where it is satisfied | Evidence command |
|---|---|---|
| 1. Layout matches §3.1; all Cowork-authored files present and byte-identical apart from reported typo fixes | Task 1 (content commit) + Task 6 Step 6 | `find . -type f`, `sha256sum -c /tmp/cowork-authored.sha256` |
| 2. `marketplace.json` and both `plugin.json` are valid JSON and pass `claude plugin validate` | Task 2 Steps 1, 2–4 (JSON parse), 5 (validator), 6 (contract) | `claude plugin validate .` + the two plugin paths |
| 3. `grep -ri "chatrevenue\|sasha" plugins/` returns nothing | Task 1 Step 3, Task 5 Step 4, Task 6 Step 6 — and DP1 keeps the owner email out of `plugins/` | `grep -rin … ; echo exit=$?` → `exit=1` |
| 4. Every `references/*.md` named in a `SKILL.md` exists at that path | Task 1 Step 2, re-run in Task 6 Step 6 | the per-`SKILL.md` reference loop |
| 5. Local `marketplace add ./` + `install delivery-executor@digiteam` succeed; the skill appears | Task 6 Steps 2–5 | `claude plugin marketplace add ./`, `claude plugin install`, `claude plugin list`, the installed `SKILL.md` path |
| 6. GitHub repo public, `main` pushed, README renders with both connect paths | Task 4 (content) + Task 7 Steps 2, 4, 6 | `gh repo view --json visibility,defaultBranchRef`, `gh api …/readme` |
| 7. The four stage folders exist and the plan sits in the folder matching the reported phase | phase 0b (folders + plan in `drafts/`), Task 6 Step 6 (`ongoing/`), Task 8 Step 5 (`done/`) | `ls engineering_plans/*/` |
| §8 integration inputs (Cowork's phase 5, read-only) | Task 2 Step 6 pre-checks the same `source`/`plugin.json`/`SKILL.md` contract; Task 7 Steps 4–5 prove public reachability; DP4 records the one observed protocol delta | the contract script, `gh api …/contents/.claude-plugin/marketplace.json` |

## Phase boundaries

- **Phase 0b (already done in this handoff):** `git init -b main`, the four stage folders with `.gitkeep`, and one commit holding `design_docs/`, `engineering_plans/` and this plan. Nothing under `plugins/` is staged.
- **Phase 2 (Tasks 1–6):** first `git mv` this plan `drafts → ongoing` and commit that move, then build and validate everything locally. No GitHub, no remote, no tag.
- **Phase 4 (Tasks 7–8):** create the public repo, push `main`, tag `v0.1.0`, verify, then `git mv` the plan `ongoing → done` and commit.
