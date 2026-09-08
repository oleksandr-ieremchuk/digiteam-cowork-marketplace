# Deterministic executor triggering (`executor-trigger`) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship `v0.1.1` of both DigiTeam plugins so that every Delivery HandOff starts with `/delivery-executor:executing-delivery-handoff` and therefore invokes the executor skill deterministically, instead of relying on description matching.

**Architecture:** No application code. Cowork has already rewritten the four skill files that carry the protocol text (see the byte-identity baseline below); this plan **commits them as-is**, then does the three mechanical jobs around them — bump the shared version in five places, add the README notes of spec §3.2.2, and prove the change actually works by installing the plugin from a local marketplace and firing a real HandOff at a fresh interactive `claude` session. "Tests" here are the manifest validator, greps, a parity script, and that live trigger test — each run *before* the change (expect the old state) and *after* (expect the new one).

**Tech Stack:** JSON manifests (Claude plugin/marketplace schema), Markdown, `claude` CLI 2.1.118 (`plugin validate|marketplace add|remove|install|uninstall|list`), `python` 3 (parity + contract scripts), `git` 2.46.0.windows.1, `gh` 2.76.2 (authenticated as `oleksandr-ieremchuk`, ssh git protocol).

**Spec:** [`design_docs/executor-trigger-design.md`](../../design_docs/executor-trigger-design.md)

## Global Constraints

- **Release version is `0.1.1`** — exactly five `version` occurrences change (spec §3.2.1, D5). Both plugins bump together even though only some files changed (ADR 0005: one shared version).
- **Tag is plain `v0.1.1`** on the verified commit, phase 4 only (spec §3.2.4). Do not use `claude plugin tag` — it produces `{name}--v{version}`.
- **The four Cowork-authored skill files are commit-as-is** (spec §3.1). Do not rewrite, reflow, reword or "improve" them. A typo fix is permitted but must be reported as a deviation with the exact before/after.
- **Byte-identity is checked by sha256** against the baseline table below. A hash that does not match at commit time means the file was touched — stop and report.
- **LF line endings** everywhere (ADR 0006, enforced by `.gitattributes`).
- **No `chatrevenue` / `sasha` anywhere under `plugins/`** (spec §4.3).
- **The Report block stays byte-identical** between the two `report-format.md` files (spec §4.4; §3.3 puts the Report explicitly out of scope for this feature).
- **Nothing else in the manifests changes** (spec §3.2.1) — only the `version` values.
- **No push, no tag, no GitHub anything before Cowork's phase-3 deploy-approval gate.**

## Byte-identity baseline (spec §3.1 plus the spec itself)

Recorded 2026-09-08 from the working folder, before any commit. Five files: the spec (committed in phase 0b together with this plan) and the four skill files Cowork edited in place (committed in phase 2, Task 1).

| sha256 | File | Committed in |
|---|---|---|
| `75e2c2a9802e40ba9c4d53242880b533584a3956e592f0362213805bd312b458` | `design_docs/executor-trigger-design.md` | phase 0b |
| `b6b1880084b6fb9a4e1fcbf344e7ef1a454b3ba20a3a7da78e582155c8fd0eb5` | `plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md` | phase 2, Task 1 |
| `e486e641dafdfa7d302643bc05422e2e5c271ff15223733f0ddff4a510c8510f` | `plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md` | phase 2, Task 1 |
| `666db361a6a371072f84dae245c9b89142f938f899d9a4194d8d8a6a5de193bd` | `plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md` | phase 2, Task 1 |
| `6b8af3ecb5e97187bb0fdb2f5452e308778b2e70b68edbc05099d221d01be907` | `plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md` | phase 2, Task 1 |

Re-check at any time with:

```bash
sha256sum design_docs/executor-trigger-design.md \
  plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md \
  plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md \
  plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md \
  plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md
```

## File structure

| File | Responsibility | Change in this plan |
|---|---|---|
| the four files above | the HandOff protocol text on both sides | **none** — committed byte-identical (Task 1) |
| `.claude-plugin/marketplace.json` | marketplace manifest | three `version` values `0.1.0 → 0.1.1` (Task 2) |
| `plugins/delivery-orchestrator/.claude-plugin/plugin.json` | orchestrator manifest | one `version` value (Task 2) |
| `plugins/delivery-executor/.claude-plugin/plugin.json` | executor manifest | one `version` value (Task 2) |
| `README.md` | marketplace front door | two sentences in "Connect to Claude Code", one note in "Versioning" (Task 3) |
| `engineering_plans/{drafts,ongoing,done}/executor-trigger-plan.md` | this plan; the durable phase signal | `drafts → ongoing` (Task 1) → `done` (Task 7) |

**Deliberately unchanged**, so a reviewer does not read the omission as a miss:
`plugins/delivery-executor/README.md` still describes the skill's trigger phrases without the slash line — spec §3.2.2 scopes the README work to the **root** README only. Both `report-format.md`, both `phase-map.md` and all other orchestrator references are unchanged on purpose (spec §3.1). `design_docs/architecture/references/delivery-protocol.md` and ADR 0007 are **phase 6, Cowork's** (spec §3.2.3) — not this plan's.

## Phase discipline

- **Phase 2 = Tasks 1–5.** Move this plan `drafts → ongoing` and commit that move first, then commit content, bump versions, add the README notes, validate, and run the live trigger test locally. **No GitHub, no push, no tag.** The plan ends phase 2 in `ongoing/`; Cowork's gate 3 (change review + deploy approval) comes next.
- **Phase 4 = Tasks 6–7.** Only after Cowork approves deploy: push `main`, verify the remote install round trip at `0.1.1`, tag `v0.1.1` on the verified commit, push the tag, then `git mv` the plan `ongoing → done` and commit.

---

# PHASE 2 — commit, bump, document, verify locally (Tasks 1–5)

### Task 1: Move the plan to `ongoing` and commit Cowork's four files as-is

**Files:**
- Move: `engineering_plans/drafts/executor-trigger-plan.md` → `engineering_plans/ongoing/executor-trigger-plan.md`
- Commit unmodified: the four skill files from the baseline table
- Test: `sha256sum` against the baseline, `git status --porcelain`, `git show --stat`

**Interfaces:**
- Consumes: the working-folder edits Cowork left in place; the spec, already committed in phase 0b.
- Produces: a clean tree carrying the `0.1.1` protocol text, so Tasks 2–5 can validate and install a coherent plugin. Task 5's trigger test depends on the executor `SKILL.md` and `handoff-format.md` committed here.

- [ ] **Step 1: Move the plan `drafts → ongoing` and commit that move on its own**

The stage folder is what Cowork gates on, so this move is its own commit, before any content work.

```bash
git mv engineering_plans/drafts/executor-trigger-plan.md engineering_plans/ongoing/executor-trigger-plan.md
git commit -m "$(printf '%s\n' 'plan(executor-trigger): drafts → ongoing' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

Expected: one commit touching one renamed file. Verify with `git show --stat HEAD` → `rename engineering_plans/{drafts => ongoing}/executor-trigger-plan.md`.

- [ ] **Step 2: Confirm the four files still match the baseline before staging**

This is the test that guards spec §4.1. Run it *before* `git add`, so a stray edit is caught while it is still cheap.

```bash
sha256sum plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md \
  plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md \
  plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md \
  plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md
```

Expected, in this order:

```
b6b1880084b6fb9a4e1fcbf344e7ef1a454b3ba20a3a7da78e582155c8fd0eb5 *.../references/handoff-prompt.md
e486e641dafdfa7d302643bc05422e2e5c271ff15223733f0ddff4a510c8510f *.../orchestrating-delivery/SKILL.md
666db361a6a371072f84dae245c9b89142f938f899d9a4194d8d8a6a5de193bd *.../references/handoff-format.md
6b8af3ecb5e97187bb0fdb2f5452e308778b2e70b68edbc05099d221d01be907 *.../executing-delivery-handoff/SKILL.md
```

If any hash differs: **stop**. Do not "restore" the file from memory. Report the mismatching path under `Blockers` — Cowork owns that content and will re-issue it.

- [ ] **Step 3: Read the diff once, to catch a typo worth reporting**

```bash
git diff --stat
git diff
```

Expected: four files changed, no other paths. Read for typos only. If you find one, fix exactly that character span, re-run Step 2, record the new hash, and list the before/after under `Deviations` in the Report — a typo fix is the *only* permitted edit (spec §3.1).

- [ ] **Step 4: Verify the slash line actually landed in both HandOff blocks**

The whole point of the feature, checked mechanically before it is committed:

```bash
python - <<'EOF'
import re
def first_block(p):
    return re.search(r'^```\n(.*?)^```', open(p, encoding='utf-8').read(), re.S | re.M).group(1)
SLASH = '/delivery-executor:executing-delivery-handoff'
for label, path in [
    ('orchestrator template', 'plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md'),
    ('executor shape',        'plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md'),
]:
    line1 = first_block(path).splitlines()[0]
    assert line1 == SLASH, (label, line1)
    print('OK', label, '-> line 1 is the slash command')
EOF
```

Expected: `OK orchestrator template …` and `OK executor shape …`.

- [ ] **Step 5: Verify D2 and D4 — still model-invocable, and the old `Do:` bullet is gone**

D2 says the skill keeps its plain-paste path: the slash line is a guarantee on top, not a replacement, so the frontmatter must **not** have acquired `disable-model-invocation`. D4 says the redundant bullet is dropped from both sides.

```bash
grep -rn "disable-model-invocation" plugins/ ; echo "D2-grep exit=$?"
grep -rn "Run this HandOff with" plugins/ ; echo "D4-grep exit=$?"
```

Expected: no matches for either, `D2-grep exit=1` and `D4-grep exit=1`.

- [ ] **Step 6: Commit the four files, and only those four**

```bash
git add plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md \
        plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md \
        plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md \
        plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md
git status --porcelain
```

Expected: exactly four staged modifications, nothing unstaged, nothing untracked.

```bash
git commit -m "$(printf '%s\n' 'feat(handoff): make the executor slash command the first line of every HandOff' '' 'Auto-invocation by description matching never fired in the marketplace-bootstrap' 'run (0/3). The HandOff template and shape now open with' '/delivery-executor:executing-delivery-handoff, which invokes the skill' 'deterministically and passes the rest as its argument (D1). The skill stays' 'model-invocable (D2) and handles an empty argument by asking (D3); the' 'redundant "Run this HandOff with…" bullet is dropped (D4).' '' 'Content authored by Cowork, committed byte-identical.' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

- [ ] **Step 7: Confirm the commit is byte-identical to the baseline**

```bash
git show --stat HEAD
for f in plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md \
         plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md \
         plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md \
         plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md; do
  printf '%s  %s\n' "$(git show "HEAD:$f" | sha256sum | cut -d' ' -f1)" "$f"
done
```

Expected: four files in the commit; the four hashes equal the baseline table. This is the evidence line for spec §4.1.

---

### Task 2: Bump all five `version` occurrences to `0.1.1`

**Files:**
- Modify: `.claude-plugin/marketplace.json:9` (`metadata.version`), `:16` (orchestrator entry), `:24` (executor entry)
- Modify: `plugins/delivery-orchestrator/.claude-plugin/plugin.json:3`
- Modify: `plugins/delivery-executor/.claude-plugin/plugin.json:3`
- Test: the version grep, `json.load`, `claude plugin validate` ×3, the contract script

**Interfaces:**
- Consumes: nothing from Task 1 — independent, but sequenced after it so the trigger test in Task 5 installs a plugin whose manifest and content agree.
- Produces: `version == "0.1.1"` in all five places, which Task 5 reads back out of `claude plugin list` and Task 6 re-checks from the GitHub remote.

- [ ] **Step 1: Run the failing check — five occurrences still read `0.1.0`**

```bash
grep -rn '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
grep -rc '"version": "0.1.1"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
```

Expected (the "before" state): five lines all showing `0.1.0`; the second grep prints `…:0` for all three files.

- [ ] **Step 2: Bump the three occurrences in `marketplace.json`**

All three are the string `"version": "0.1.0"`; nothing else in the file contains that string, so a whole-file replace is exact and touches nothing else (spec §3.2.1). `newline=''` preserves the existing LF endings.

```bash
python - <<'EOF'
p = '.claude-plugin/marketplace.json'
s = open(p, encoding='utf-8', newline='').read()
assert s.count('"version": "0.1.0"') == 3, s.count('"version": "0.1.0"')
open(p, 'w', encoding='utf-8', newline='').write(s.replace('"version": "0.1.0"', '"version": "0.1.1"'))
print('marketplace.json: 3 replaced')
EOF
```

Expected: `marketplace.json: 3 replaced`.

- [ ] **Step 3: Bump the two `plugin.json` files**

```bash
python - <<'EOF'
for p in ('plugins/delivery-orchestrator/.claude-plugin/plugin.json',
          'plugins/delivery-executor/.claude-plugin/plugin.json'):
    s = open(p, encoding='utf-8', newline='').read()
    assert s.count('"version": "0.1.0"') == 1, (p, s.count('"version": "0.1.0"'))
    open(p, 'w', encoding='utf-8', newline='').write(s.replace('"version": "0.1.0"', '"version": "0.1.1"'))
    print(p, ': 1 replaced')
EOF
```

Expected: one `: 1 replaced` line per file.

- [ ] **Step 4: Run the check again — it must now pass**

```bash
grep -rn '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
grep -rn '0\.1\.0' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json ; echo "stale-grep exit=$?"
git diff --stat
```

Expected: five lines all reading `"version": "0.1.1"`; `stale-grep exit=1` with no matches; `git diff --stat` shows exactly three files, five insertions, five deletions.

- [ ] **Step 5: Verify the JSON is still valid and the manifests still agree**

This is the same contract Cowork re-runs at the phase-5 gate, with the version assertion moved to `0.1.1`:

```bash
python - <<'EOF'
import json, os
m = json.load(open('.claude-plugin/marketplace.json'))
assert m['name'] == 'digiteam', m['name']
assert m['metadata']['version'] == '0.1.1', m['metadata']['version']
for entry in m['plugins']:
    src = entry['source']
    pj = json.load(open(os.path.join(src, '.claude-plugin', 'plugin.json')))
    assert pj['name'] == entry['name'], (pj['name'], entry['name'])
    assert pj['description'] == entry['description'], entry['name']
    assert pj['version'] == entry['version'] == '0.1.1', (pj['version'], entry['version'])
    assert 'dependencies' not in pj, entry['name']
    print('OK', entry['name'], '->', src, pj['version'])
print('contract OK at 0.1.1')
EOF
```

Expected: two `OK …0.1.1` lines and `contract OK at 0.1.1`.

- [ ] **Step 6: Run the validator on all three manifest roots**

```bash
claude plugin validate .
claude plugin validate ./plugins/delivery-orchestrator
claude plugin validate ./plugins/delivery-executor
```

Expected: PASS, three times, no errors. Paste the actual output into the Report — this is half the evidence for spec §4.2.

- [ ] **Step 7: Commit**

```bash
git add .claude-plugin/marketplace.json plugins/delivery-orchestrator/.claude-plugin/plugin.json plugins/delivery-executor/.claude-plugin/plugin.json
git commit -m "$(printf '%s\n' 'chore(release): bump both plugins and the marketplace to 0.1.1' '' 'Five version occurrences: metadata.version plus both plugin entries in' 'marketplace.json, and each plugin.json. One shared version per ADR 0005 (D5).' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

---

### Task 3: README — the slash-line note and the versioning note (spec §3.2.2)

**Files:**
- Modify: `README.md` — insert after the shell install block (currently `README.md:53`), and extend the "Versioning" section (currently `README.md:127-132`)
- Verify unchanged: the "Connect to Cowork" re-add sentence (`README.md:38-39`) is already present — **keep it, do not duplicate it**
- Test: greps for the required points, plus a duplicate guard

**Interfaces:**
- Consumes: the slash command string introduced in Task 1 and the `0.1.1` version from Task 2 — the README must not advertise a version or a command the repo does not ship.
- Produces: the install-time explanation a user needs when the command is rejected; nothing downstream depends on it programmatically.

- [ ] **Step 1: Run the failing checks — neither new point is in the README yet**

```bash
grep -n "delivery-executor:executing-delivery-handoff" README.md ; echo "slash-note exit=$?"
grep -n "v0.1.1" README.md ; echo "version-note exit=$?"
grep -c "uninstall and" README.md
```

Expected (the "before" state): `slash-note exit=1` and `version-note exit=1` (both absent), and `1` — the Cowork re-add sentence already exists exactly once, which is why §3.2.2 says keep it rather than add it.

- [ ] **Step 2: Add the two sentences to "Connect to Claude Code"**

They go after the shell install block and before the "For a whole team" paragraph, so the reader meets the slash line right after installing the thing that answers it.

```bash
python - <<'EOF'
p = 'README.md'
s = open(p, encoding='utf-8', newline='').read()
anchor = "For a whole team, commit this to the repo's `.claude/settings.json` instead, so\n"
assert s.count(anchor) == 1, s.count(anchor)
note = (
    "Every Delivery HandOff Cowork produces begins with the line\n"
    "`/delivery-executor:executing-delivery-handoff`, so pasting the whole block\n"
    "into Claude Code as a single message runs the executor on it directly. If\n"
    "Claude Code reports that command as unknown, the plugin is not installed — run\n"
    "the two commands above.\n\n"
)
open(p, 'w', encoding='utf-8', newline='').write(s.replace(anchor, note + anchor))
print('slash note inserted')
EOF
```

Expected: `slash note inserted`.

- [ ] **Step 3: Extend the "Versioning" section**

```bash
python - <<'EOF'
p = 'README.md'
s = open(p, encoding='utf-8', newline='').read()
anchor = "the repo `vX.Y.Z` and push the tag.\n"
assert s.count(anchor) == 1, s.count(anchor)
note = (
    "\n`v0.1.1` is the first bump after the initial release. Both plugins move\n"
    "together even when only one of them changed, so all five `version`\n"
    "occurrences change on every release.\n"
)
open(p, 'w', encoding='utf-8', newline='').write(s.replace(anchor, anchor + note))
print('versioning note appended')
EOF
```

Expected: `versioning note appended`.

- [ ] **Step 4: Run the checks again — they must now pass, without duplication**

```bash
grep -n "delivery-executor:executing-delivery-handoff" README.md
grep -n "v0.1.1" README.md
grep -c "uninstall and" README.md
git diff README.md
```

Expected: the slash command appears once, in "Connect to Claude Code"; `v0.1.1` appears once, in "Versioning"; `uninstall and` still prints `1` — the Cowork sentence was neither duplicated nor removed. Read the diff: it must touch only those two regions.

- [ ] **Step 5: Confirm the README renders as intended**

```bash
sed -n '41,62p' README.md
sed -n '/^## Versioning/,/^## License/p' README.md
```

Expected: the new paragraph sits between the shell block and "For a whole team"; the versioning note is the last paragraph before "## License". Blank lines around both — no run-on paragraphs.

- [ ] **Step 6: Commit**

```bash
git add README.md
git commit -m "$(printf '%s\n' 'docs(readme): explain the HandOff slash line and the 0.1.1 bump' '' 'Connect to Claude Code now says every HandOff opens with' '/delivery-executor:executing-delivery-handoff and what an unknown-command' 'error means. Versioning notes that 0.1.1 is the first bump and that both' 'plugins move together.' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
```

---

### Task 4: Full static verification sweep (spec §4.1–§4.4)

**Files:**
- Modify: none — this task only reads
- Test: sha256 baseline, `claude plugin validate` ×3, the version grep, the generic-name grep, the parity script

**Interfaces:**
- Consumes: everything Tasks 1–3 committed.
- Produces: the pasteable evidence block for acceptance criteria §4.1–§4.4 in the Report. Task 5 assumes this passed.

- [ ] **Step 1: §4.1 — the four files are committed byte-identical**

```bash
for f in plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md \
         plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md \
         plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md \
         plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md; do
  printf '%s  %s\n' "$(git show "HEAD:$f" | sha256sum | cut -d' ' -f1)" "$f"
done
```

Expected: the four baseline hashes, unless a typo fix was reported in Task 1 Step 3.

- [ ] **Step 2: §4.2 — validator passes and all five versions read `0.1.1`**

```bash
claude plugin validate . \
  && claude plugin validate ./plugins/delivery-orchestrator \
  && claude plugin validate ./plugins/delivery-executor
grep -rn '"version"' .claude-plugin/marketplace.json plugins/*/.claude-plugin/plugin.json
```

Expected: three PASSes; five lines, all `0.1.1`.

- [ ] **Step 3: §4.3 — no company or personal names under `plugins/`**

```bash
grep -rin "chatrevenue\|sasha" plugins/ ; echo "generic-grep exit=$?"
```

Expected: no output, `generic-grep exit=1`.

- [ ] **Step 4: §4.4 — parity between the two sides**

Three assertions in one script: the Report block is byte-identical, both HandOff blocks open with the slash line, and both carry the same fields in the same order with `Fix:` as the documented executor-only exception.

```bash
python - <<'EOF'
import re, hashlib
SLASH = '/delivery-executor:executing-delivery-handoff'
ORCH = 'plugins/delivery-orchestrator/skills/orchestrating-delivery/references/'
EXEC = 'plugins/delivery-executor/skills/executing-delivery-handoff/references/'

def first_block(p):
    return re.search(r'^```\n(.*?)^```', open(p, encoding='utf-8').read(), re.S | re.M).group(1)

# 1. Report block byte-identical
o_rep, e_rep = first_block(ORCH + 'report-format.md'), first_block(EXEC + 'report-format.md')
assert o_rep == e_rep, 'Report blocks diverged'
print('OK  report block identical, sha256', hashlib.sha256(o_rep.encode()).hexdigest()[:16])

# 2. Both HandOff blocks start with the slash line
o_ho, e_ho = first_block(ORCH + 'handoff-prompt.md'), first_block(EXEC + 'handoff-format.md')
assert o_ho.splitlines()[0] == SLASH, o_ho.splitlines()[0]
assert e_ho.splitlines()[0] == SLASH, e_ho.splitlines()[0]
print('OK  both HandOff blocks open with the slash line')

# 3. Same fields, same order; Fix: is the executor-only exception
def fields(b):
    return [l.split(':')[0] for l in b.splitlines() if re.match(r'^[A-Z][A-Za-z ]+:', l)]
o_f, e_f = fields(o_ho), fields(e_ho)
assert [f for f in e_f if f != 'Fix'] == o_f, (o_f, e_f)
assert 'Fix' in e_f and 'Fix' not in o_f, (o_f, e_f)
print('OK  fields match in order:', o_f)
print('OK  Fix: present only in the executor shape, as documented')
print('parity OK')
EOF
```

Expected: four `OK` lines and `parity OK`. This script was run read-only during planning against the current working folder and passed (report block sha256 `f4de91d249b37067`, fields `['Spec', 'Your scope in this repo', 'Phase to run', 'Gate already passed', 'Do', 'Acceptance criteria for this repo', 'Report back']`) — a failure here means Task 1 committed something other than what was reviewed.

- [ ] **Step 5: Confirm the tree is clean and the history is the expected commits**

```bash
git status --porcelain ; echo "status exit=$?"
git log --oneline -5
```

Expected: empty status; the four commits `plan(executor-trigger): drafts → ongoing`, `feat(handoff): …`, `chore(release): …`, `docs(readme): …` sitting on top of the phase-0b commit. No commit to make in this task — it is verification only.

---

### Task 5: The trigger test — local install, live HandOff, cleanup (spec §4.5)

This is the point of the feature. Everything before it is bookkeeping; this task is the only thing that proves the problem in spec §1 is actually solved.

**Files:**
- Modify: none in the repo — the changes are to the local `claude` plugin install, and are undone in Step 8
- Test: `claude plugin marketplace add ./`, `claude plugin install`, `claude plugin list`, a real interactive `claude` session, `git status --porcelain` before and after

**Interfaces:**
- Consumes: the committed executor `SKILL.md` and `handoff-format.md` from Task 1, and the `0.1.1` manifests from Task 2 (`claude plugin list` must report `0.1.1`, proving the installed copy is the new one and not a cached `0.1.0`).
- Produces: the recorded answer to the open question in spec §4.5 — did the whole multi-line block arrive as the skill's argument, or was it collapsed. That answer decides whether D1 stands as written or D3 becomes the documented path and Cowork issues a corrective HandOff.

- [ ] **Step 1: Start from a known-clean plugin state**

A stale `digiteam` marketplace, or a `delivery-executor` installed from GitHub at `0.1.0`, would make the test meaningless — you would be firing at the old skill.

```bash
claude plugin list
claude plugin marketplace list
```

Expected: note what is there. If `delivery-executor` is already installed (likely, from the `v0.1.0` GitHub round trip), remove it and its marketplace before continuing:

```bash
claude plugin uninstall delivery-executor
claude plugin marketplace remove digiteam
claude plugin list
```

Expected after cleanup: no `delivery-executor`, no `digiteam` marketplace.

- [ ] **Step 2: Record the pre-test tree state**

Criterion (c) is "does not touch the tree", which needs a before to compare against.

```bash
git status --porcelain > "$TMPDIR/pre-trigger-status.txt"
git rev-parse HEAD
cat "$TMPDIR/pre-trigger-status.txt" ; echo "pre-status lines: $(wc -l < "$TMPDIR/pre-trigger-status.txt")"
```

Expected: `pre-status lines: 0` and the HEAD from Task 4 Step 5. (If `$TMPDIR` is unset in your shell, use the session scratchpad directory instead — just keep both snapshots in the same place.)

- [ ] **Step 3: Install the executor from the local marketplace at `0.1.1`**

```bash
claude plugin marketplace add ./
claude plugin install delivery-executor@digiteam
claude plugin list
```

Expected: the marketplace adds as `digiteam`; `delivery-executor` installs; `claude plugin list` shows it at **`0.1.1`**. If it shows `0.1.0`, the CLI served a cached copy — remove the marketplace, re-add, reinstall, and only continue once the list reads `0.1.1`.

- [ ] **Step 4: Confirm the installed skill is the new content, not a cache**

```bash
INSTALLED=$(find ~/.claude/plugins -path '*executing-delivery-handoff/SKILL.md' 2>/dev/null | head -1)
echo "$INSTALLED"
sha256sum "$INSTALLED"
grep -c 'ARGUMENTS' "$INSTALLED"
grep -c 'delivery-executor:executing-delivery-handoff' "$INSTALLED"
```

Expected: the hash equals `6b8af3ecb5e97187bb0fdb2f5452e308778b2e70b68edbc05099d221d01be907` (the baseline executor `SKILL.md`); both grep counts are at least `1`. A different hash means an old copy is installed — go back to Step 3.

- [ ] **Step 5: Open a *fresh interactive* `claude` session and paste the sample HandOff**

Not a subagent, not this session, not `claude -p`: a new interactive session started in this repo, so the slash-command path is exercised exactly as a real user hits it.

```bash
claude
```

Then paste **this exact text** as one message. It is a real HandOff whose `Phase to run` is deliberately mismatched: phase 2 requires the plan to be in `drafts/`, and there is no plan named `trigger-probe` in any stage folder — so a correctly behaving executor must block at precondition 3. The `Spec:` path is the real spec, so the block is reached via the plan-stage check rather than a missing file. `MARKER-7Q4X` is a string that appears nowhere else in the repo: if the session's output repeats it, the whole block reached the skill.

```
/delivery-executor:executing-delivery-handoff
Delivery HandOff — trigger-probe — repo: digiteam-cowork-marketplace

Spec: design_docs/executor-trigger-design.md
Your scope in this repo: nothing — this is a trigger probe, make no changes.
Phase to run: 2 execute
Gate already passed: none — first handoff

Do:
- Verify the preconditions and stop. Make no edits and no commits.

Acceptance criteria for this repo:
- MARKER-7Q4X is echoed back so the sender knows the whole block arrived.
- No file in the working tree is modified.

When done, print a Report and I'll paste it back to Cowork.

Report back:
- Phase completed and current plan stage (drafts/ongoing/done/documented)
- Blockers, if any
```

- [ ] **Step 6: Record the four observations, verbatim**

Write these into the Report exactly as observed — this is the data the spec asks for, and a negative result is a valid, useful result.

| # | Observation | What counts as pass |
|---|---|---|
| (a) | Was the skill invoked with **no extra prompting**? | The session announces `executing-delivery-handoff` (or walks its steps) immediately, without the user asking again. |
| (b) | Did the **whole block** reach it? | Its output names the slug `trigger-probe`, this repo, phase `2`, **and** `MARKER-7Q4X`. |
| (c) | Did it **stop and touch nothing**? | It prints `Phase completed: none — blocked` with the stage mismatch under `Blockers`. |
| (d) | Argument **complete or collapsed**? | Complete = (b) holds. Collapsed = the skill ran but reports an empty or partial HandOff and asks for it (D3), or only the first line arrived. |

Then, from a shell, confirm (c) against the pre-test snapshot:

```bash
git status --porcelain > "$TMPDIR/post-trigger-status.txt"
diff "$TMPDIR/pre-trigger-status.txt" "$TMPDIR/post-trigger-status.txt" && echo "TREE UNTOUCHED"
git rev-parse HEAD
```

Expected: `TREE UNTOUCHED` and the same HEAD as Step 2.

- [ ] **Step 7: If — and only if — the paste was collapsed, run the D3 fallback**

Skip this step entirely when (d) is "complete". If the client collapsed the multi-line paste, the spec's fallback must be shown to work before the feature can be called done.

In a **second** fresh interactive session, send the command alone as the first message:

```
/delivery-executor:executing-delivery-handoff
```

Expected per D3: the skill is invoked, finds its argument empty, **asks for the HandOff and stops** — it must not guess, and must not start scanning the repo. Then paste the sample block from Step 5 (without the slash line) as the next message, and re-check (b) and (c) from the table above.

Record: whether the empty-argument branch behaved as D3 specifies, and whether the two-message path produced the same blocked Report. Per spec §4.5 a collapsed paste means **D3 becomes the documented path** and Cowork amends the `handoff-prompt.md` wording in a corrective loop — flag that under `Blockers` in the Report rather than editing the wording here, since that file is Cowork's (spec §3.1).

- [ ] **Step 8: Clean up the local install**

Required by spec §4.5 — the local marketplace must not outlive the test, or the next real HandOff runs against a `./`-sourced plugin instead of the published one.

```bash
claude plugin uninstall delivery-executor
claude plugin marketplace remove digiteam
claude plugin list
claude plugin marketplace list
git status --porcelain ; echo "status exit=$?"
```

Expected: no `delivery-executor`, no `digiteam` marketplace, empty `git status`. Nothing to commit in this task.

**Phase 2 ends here.** The plan stays in `engineering_plans/ongoing/`. Print the Report and stop — pushing or tagging now would jump Cowork's gate 3.

---

# PHASE 4 — release (Tasks 6–7) — only after deploy approval

### Task 6: Push `main` and verify the remote install round trip at `0.1.1`

**Files:**
- Modify: none — this task publishes what phase 2 committed
- Test: `git push`, `claude plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace`, `claude plugin install`, `claude plugin list`, `gh api`

**Interfaces:**
- Consumes: the four phase-2 commits on local `main`, and Cowork's phase-3 deploy approval.
- Produces: the pushed commit that Task 7 tags. The tag must point at a commit that has *already* passed this round trip (spec §3.2.4).

- [ ] **Step 1: Confirm the gate and the starting state**

```bash
git status --porcelain ; echo "status exit=$?"
git log --oneline -5
git log --oneline origin/main -1
```

Expected: clean tree; local `main` ahead of `origin/main` by the phase-0b commit plus the four phase-2 commits. Do not start this task without Cowork's recorded deploy approval in the HandOff's `Gate already passed`.

- [ ] **Step 2: Push `main`**

```bash
git push origin main
git log --oneline origin/main -1
```

Expected: the push succeeds and `origin/main` now points at the `docs(readme): …` commit.

- [ ] **Step 3: Verify the remote round trip from a clean plugin state**

Same shape as the `v0.1.0` round trip, asserting `0.1.1` this time.

```bash
claude plugin marketplace list
claude plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace
claude plugin install delivery-executor@digiteam
claude plugin list
```

Expected: the GitHub-sourced marketplace adds cleanly; `delivery-executor` installs; `claude plugin list` reports **`0.1.1`**. If it reports `0.1.0`, the push did not land or a cache is serving the old manifest — resolve before tagging.

- [ ] **Step 4: Verify the published manifest reads `0.1.1`**

```bash
gh api repos/oleksandr-ieremchuk/digiteam-cowork-marketplace/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d | grep -n '"version"'
```

Expected: three lines, all `0.1.1`.

- [ ] **Step 5: Clean up the remote-sourced install**

```bash
claude plugin uninstall delivery-executor
claude plugin marketplace remove digiteam
claude plugin list
```

Expected: no leftovers. (Sasha reinstalls for real at phase 5, spec §5.)

---

### Task 7: Tag `v0.1.1` and move the plan `ongoing → done`

**Files:**
- Move: `engineering_plans/ongoing/executor-trigger-plan.md` → `engineering_plans/done/executor-trigger-plan.md`
- Test: `git tag -l`, `git ls-remote --tags`, `git rev-parse`, `git status`

**Interfaces:**
- Consumes: Task 6's verified, pushed commit.
- Produces: the `v0.1.1` tag that Cowork's phase-5 integration verification fetches (spec §5), and the `done/` stage folder that tells Cowork this repo is ready for the acceptance gate.

- [ ] **Step 1: Confirm Task 6 passed and no `v0.1.1` tag exists**

```bash
git tag -l
git ls-remote --tags origin
```

Expected: `v0.1.0` only, locally and remotely. The tag goes on **after** the round trip passes — if anything in Task 6 failed, stop and report instead of tagging.

- [ ] **Step 2: Tag the verified commit**

```bash
git tag -a v0.1.1 -m "digiteam marketplace v0.1.1 — HandOffs open with the executor slash command"
git push origin v0.1.1
```

- [ ] **Step 3: Verify the tag landed on the right commit**

```bash
git rev-parse v0.1.1^{commit} && git rev-parse origin/main
git ls-remote --tags origin
```

Expected: `v0.1.1^{commit}` and `origin/main` resolve to the same SHA; the tag is listed remotely.

- [ ] **Step 4: Move the plan `ongoing → done` and commit**

```bash
git mv engineering_plans/ongoing/executor-trigger-plan.md engineering_plans/done/executor-trigger-plan.md
git commit -m "$(printf '%s\n' 'plan(executor-trigger): ongoing → done' '' 'Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>')"
git push origin main
```

- [ ] **Step 5: Verify the final state**

```bash
ls engineering_plans/*/
git status --porcelain ; echo "status exit=$?"
git log --oneline -2
```

Expected: `executor-trigger-plan.md` in `done/` and nowhere else; clean tree; the stage-move commit pushed. Note that the stage-move commit sits *after* the tag — that is correct and matches `v0.1.0`: the tag marks the released content, not the bookkeeping.

**Phase 4 ends here.** Print the Report; Cowork's gate 5 (acceptance + integration verification) is next.

---

## Acceptance criteria → steps

| Spec §4 criterion | Implemented / verified by | Evidence command |
|---|---|---|
| 1. Four Cowork files committed byte-identical (sha256) | Task 1 Steps 2, 6, 7; Task 4 Step 1 | `git show HEAD:<path> \| sha256sum` against the baseline table |
| 2. `claude plugin validate` passes on all three roots; five `version` occurrences read `0.1.1` | Task 2 Steps 4–6; Task 4 Step 2 | `claude plugin validate .` ×3 + `grep -rn '"version"'` |
| 3. `grep -ri "chatrevenue\|sasha" plugins/` → nothing | Task 4 Step 3 | that grep, exit 1 |
| 4. Parity: Report blocks byte-identical; both HandOff blocks start with the slash line and carry the same fields in order (`Fix:` excepted) | Task 1 Step 4; Task 4 Step 4 | the parity script (already passes read-only) |
| 5. Trigger test: local install, fresh interactive session, mismatched sample HandOff — (a) invoked unprompted, (b) block arrived, (c) blocked without touching the tree; record complete-vs-collapsed argument; run the D3 fallback if collapsed; clean up | **Task 5**, Steps 1–8 | `claude plugin list` at `0.1.1`; installed `SKILL.md` sha256; the session transcript; `diff` of pre/post `git status`; the observation table in Step 6 |
| 6. Remote round trip at `0.1.1`; tag `v0.1.1` on the verified commit | Task 6 Steps 2–4; Task 7 Steps 2–3 | `claude plugin list` after the GitHub install; `git rev-parse v0.1.1^{commit}` == `origin/main` |
| 7. Plan sits in the stage folder matching the reported phase | Task 1 Step 1 (`ongoing`); Task 7 Step 4 (`done`) | `ls engineering_plans/*/` |

Also covered, from the decisions table rather than §4: **D1** (slash line is line 1 of both blocks) at Task 1 Step 4; **D2** (no `disable-model-invocation` — the plain-paste path survives) and **D4** (the old `Do:` bullet is gone) at Task 1 Step 5; **D3** (empty argument → ask and stop) at Task 5 Step 7; **D5** (one shared version, five occurrences) at Task 2.

## Open risks

- **R1 — the multi-line paste may be collapsed.** This is the one genuinely unknown outcome, and Task 5 Step 7 is its branch. A collapsed paste does not fail the plan; it promotes D3 to the documented path and hands Cowork a wording fix (spec §4.5). Report it, do not patch `handoff-prompt.md` here.
- **R2 — a cached `0.1.0` plugin would invalidate the trigger test.** Task 5 Steps 1, 3 and 4 exist only to rule this out: the installed `SKILL.md` hash must equal the baseline before the test counts.
- **R3 — the sample HandOff could be acted on rather than blocked.** Mitigated by using slug `trigger-probe`, which has no plan in any stage folder, so there is nothing for a misbehaving run to move; and by the pre/post `git status` diff in Task 5 Steps 2 and 6.
