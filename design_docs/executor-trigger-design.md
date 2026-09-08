# Deterministic executor triggering — v0.1.1

- **Slug:** `executor-trigger`
- **Status:** spec — ready for planning (phase 0b)
- **Author:** Cowork (orchestrator), 2026-09-08
- **Target repos:** `digiteam-cowork-marketplace` (this repo) — single-repo feature
- **Release:** `v0.1.1` — both plugins bump `0.1.0 → 0.1.1`

## 1. Problem

In the `marketplace-bootstrap` run, the `executing-delivery-handoff` skill never
auto-triggered from a pasted HandOff — three attempts (two phase-7 reviews, one
phase 8), zero hits. Phase 8 only ran through the skill because the user typed
`/delivery-executor:executing-delivery-handoff` explicitly. Auto-invocation is
the model matching the skill `description` against the message; a long pasted
block is evidently a weak match. The process must not depend on that.

## 2. Decisions taken (brainstorm, 2026-09-08)

| # | Decision | Alternative rejected | Why |
|---|---|---|---|
| D1 | **The first line of every HandOff is the slash command** `/delivery-executor:executing-delivery-handoff`; the block follows on the next lines. | rely on description matching; put the command in the `Do:` list (v0.1.0 did — it was ignored) | A message starting with a slash command invokes that skill deterministically and passes the rest as `$ARGUMENTS` (docs: code.claude.com/docs/en/skills). |
| D2 | Keep the skill **model-invocable** (no `disable-model-invocation`). | slash-only skill | The plain-paste path still works if someone strips the first line; the slash line is a guarantee on top, not a replacement. |
| D3 | The skill handles an **empty `$ARGUMENTS`**: ask for the HandOff and stop. | assume the block is always attached | If a multi-line paste after the command is ever collapsed by the client, the user's fallback is "type the command, paste the block as the next message" — the skill must survive that. |
| D4 | Drop the v0.1.0 `Do:` line "Run this HandOff with `executing-delivery-handoff` … if installed." | keep both | Redundant with D1; keeps the block shorter. |
| D5 | Version bump to **0.1.1** in both `plugin.json` and all three places in `marketplace.json`; tag `v0.1.1`. | bump only the changed plugin | ADR 0005: one shared version. Both plugins change anyway (template + shape). |

## 3. Scope for this repo

### 3.1 Content already authored by Cowork (commit as-is; typo fixes reported as deviations)

Cowork has edited **four** skill files in place in the working folder. Code
commits them; Code does not rewrite them.

| File | Change |
|---|---|
| `plugins/delivery-orchestrator/skills/orchestrating-delivery/references/handoff-prompt.md` | Intro paragraph rewritten (slash-first rule, fallback if plugin missing); template gains the slash line as line 1; `Do:` loses the "Run this HandOff with…" bullet. |
| `plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md` | HandOff bullet in "The Cowork ↔ Code interface" states the first line is the slash command. |
| `plugins/delivery-executor/skills/executing-delivery-handoff/references/handoff-format.md` | New paragraph on how the block arrives (`$ARGUMENTS` / plain paste / empty → ask); shape gains the slash line; `Do:` loses the bullet; field table gains a "slash line" row. |
| `plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md` | `description` mentions the slash command; Step 1 reads the block from `$ARGUMENTS` or the message, asks if both empty. |

Unchanged on purpose: both `report-format.md` (Report block stays byte-identical),
both `phase-map.md`, all other orchestrator references.

### 3.2 Code's work

1. **Manifests.** `version: "0.1.1"` in `plugins/delivery-orchestrator/.claude-plugin/plugin.json`,
   `plugins/delivery-executor/.claude-plugin/plugin.json`, and in
   `.claude-plugin/marketplace.json` (`metadata.version` + both plugin entries).
   Nothing else in the manifests changes.
2. **README.** In "Connect to Claude Code", add two sentences after the install
   commands: every HandOff from Cowork begins with
   `/delivery-executor:executing-delivery-handoff`, so pasting it as one message
   runs the executor; if the command is unknown, the plugin is not installed.
   In "Connect to Cowork" add one sentence: after a new version is published,
   remove and re-add the plugin to pick it up (already stated — keep, do not
   duplicate). In "Versioning", note that v0.1.1 is the first bump.
3. **Docs (phase 6 will author; listed here for completeness):**
   `design_docs/architecture/references/delivery-protocol.md` — the HandOff
   contract gains the slash line and the parity rule covers it; also fixes the
   one over-long source line in "Parity rule". ADR 0007 records D1–D3.
4. **Release (phase 4).** Push `main`, verify the remote install round trip at
   0.1.1, tag `v0.1.1` on the verified commit, push the tag.

### 3.3 Out of scope

- Any change to the Report block, phase map, stage folders or gates.
- Shortening the Report (backlog).
- The diff of the three reconstructed files against Sasha's original (still
  Sasha's, still open).

## 4. Acceptance criteria (gates 3 / 5)

1. The four Cowork-authored files are committed byte-identical to the working
   folder (sha256), apart from reported typo fixes.
2. `claude plugin validate .` and both plugin paths pass; all five `version`
   occurrences read `0.1.1`.
3. `grep -ri "chatrevenue\|sasha" plugins/` → nothing.
4. **Parity:** the two Report blocks remain byte-identical; the HandOff template
   (orchestrator) and shape (executor) both start with the slash line and carry
   the same fields in the same order, `Fix:` exception as before.
5. **Trigger test (the point of the feature), phase 2, functional verification:**
   install the plugin from the local marketplace (`marketplace add ./`), open a
   *fresh* interactive `claude` session in this repo, paste a sample HandOff
   whose first line is the slash command and whose `Phase to run` is a
   deliberately mismatched phase (e.g. `2 execute` while no plan exists for the
   sample slug), and confirm: (a) the skill is invoked without any extra
   prompting, (b) the block reached it (it names the sample slug and repo in
   its output), (c) it stops with `Phase completed: none — blocked` and does
   not touch the tree. Record the exact behaviour of the multi-line paste: did
   the whole block arrive in `$ARGUMENTS`, or was it collapsed? If collapsed,
   run the fallback (command alone → skill asks → paste block) and confirm it
   works; the spec then stands with D3 as the documented path and Cowork
   amends `handoff-prompt.md` wording in a corrective loop.
   Clean up the local install afterwards.
6. Remote install round trip at `0.1.1` from GitHub passes (phase 4); tag
   `v0.1.1` points at the verified commit.
7. Plan for `executor-trigger` sits in the stage folder matching the reported
   phase.

## 5. Integration verification (phase 5, Cowork, read-only)

- Fetch `marketplace.json` at tag `v0.1.1`: all versions `0.1.1`.
- Fetch both HandOff files at `v0.1.1`: first line of the template and of the
  shape is the slash command; Report blocks unchanged from `v0.1.0`.
- Sasha re-adds `delivery-orchestrator` in Cowork and `claude plugin update
  delivery-executor` (or reinstall) in Claude Code; the next real HandOff from
  the updated orchestrator starts with the slash line, and Code's Report says
  the skill triggered on its own.
