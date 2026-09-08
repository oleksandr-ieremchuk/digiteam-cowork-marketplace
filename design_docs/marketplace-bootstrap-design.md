# DigiTeam plugin marketplace — bootstrap

- **Slug:** `marketplace-bootstrap`
- **Status:** shipped v0.1.0 (gates 1/3/5 passed 2026-09-08) — documenting (phase 6)
- **Amended:** 2026-09-08 after plan review — file count (§5, §7.1), author field (§3.3), `.gitattributes` (§3.1), protocol parity (§4)
- **Author:** Cowork (orchestrator), 2026-09-08
- **Target repos:** `digiteam-cowork-marketplace` (this repo) — single-repo feature

## 1. Goal

Turn the locally-kept `orchestrating-delivery` Cowork skill into a **public
Claude plugin marketplace** that the same person (and later a team) connects to
both **Cowork** and **Claude Code**. Cowork installs the orchestrator plugin;
Claude Code installs the executor plugin; the two talk through the existing
HandOff / Report chat protocol.

This is also the first feature run through the delivery process itself, so the
repo must carry the process conventions (`design_docs/`, `engineering_plans/`
stage folders) from its first commit.

## 2. Decisions taken (brainstorm, 2026-09-08)

| # | Decision | Alternative rejected | Why |
|---|---|---|---|
| D1 | Repo is **public** on GitHub (`oleksandr-ieremchuk/digiteam-cowork-marketplace`) | private | Cowork's "Add marketplace" only accepts public GitHub repos; Claude Code accepts either. One repo serves both tools. |
| D2 | **Two plugins**: `delivery-orchestrator` (Cowork side) and `delivery-executor` (Claude Code side) | one plugin with both skills | Each tool gets only its own skill; the orchestrator's "never run git" rules don't leak into Code and vice versa. |
| D3 | Skill text is **generic** — no ChatRevenue / personal references | keep as-is | Marketplace is DigiTeam-branded and meant to be reusable; the process is fully described by its own conventions. |
| D4 | `superpowers` dependency is **documented in README**, not declared in `plugin.json` | cross-marketplace `dependencies` | superpowers lives in a third-party marketplace; a hard dependency needs an allowlist and breaks on renames. |
| D5 | Marketplace `name` is `digiteam` | `digiteam-cowork-marketplace` | It is the suffix users type: `delivery-orchestrator@digiteam`. |
| D6 | License **MIT** | none / proprietary | Public repo; nothing sensitive; simplest for reuse. Sasha can change before push. |

## 3. Scope for this repo

Everything — this is a single-repo feature. Claude Code owns all files below
except those Cowork has already authored (§5), which Code commits as-is (after
the sanity checks in §7).

### 3.1 Repository layout (target state)

```
digiteam-cowork-marketplace/
├── .claude-plugin/
│   └── marketplace.json
├── README.md
├── LICENSE                                   (MIT)
├── .gitattributes                            (`* text=auto eol=lf` — keep skill files LF on every clone)
├── design_docs/
│   └── marketplace-bootstrap-design.md       ← this spec (Cowork-authored)
├── engineering_plans/
│   ├── drafts/.gitkeep
│   ├── ongoing/.gitkeep
│   ├── done/.gitkeep
│   └── documented/.gitkeep
└── plugins/
    ├── delivery-orchestrator/
    │   ├── .claude-plugin/plugin.json
    │   ├── README.md
    │   └── skills/orchestrating-delivery/
    │       ├── SKILL.md                      ← Cowork-authored
    │       └── references/                   ← Cowork-authored (8 files)
    │           ├── architecture-doc-template.md
    │           ├── documentation-pass.md
    │           ├── handoff-prompt.md
    │           ├── integration-verification.md
    │           ├── phase-map.md
    │           ├── readme-lifecycle-amendment.md
    │           ├── report-format.md
    │           └── state-discovery.md
    └── delivery-executor/
        ├── .claude-plugin/plugin.json
        ├── README.md
        └── skills/executing-delivery-handoff/
            ├── SKILL.md                      ← Cowork-authored
            └── references/                   ← Cowork-authored (3 files)
                ├── handoff-format.md
                ├── phase-map.md
                └── report-format.md
```

### 3.2 `.claude-plugin/marketplace.json`

Follow the current schema at https://code.claude.com/docs/en/plugin-marketplaces.
Required content:

```json
{
  "name": "digiteam",
  "owner": { "name": "Oleksandr Ieremchuk", "email": "oleksandr.ieremchuk@chatrevenue.ai" },
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
```

If the schema validator rejects a field (e.g. `metadata`), drop that field and
note it as a deviation — the names, sources and descriptions are the contract.

### 3.3 `plugins/*/.claude-plugin/plugin.json`

Per https://code.claude.com/docs/en/plugins-reference. Each plugin:

- `name` exactly as in marketplace.json; `version: "0.1.0"`; `description`
  (same text as marketplace entry); `author: { "name": "Oleksandr Ieremchuk" }`
  — name only, no email, so that §7.3 holds (the email lives in
  marketplace.json `owner`, outside `plugins/`);
  `repository: "https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace"`;
  `license: "MIT"`; `keywords`.
- Skills are discovered from the default `skills/` folder — do **not** add a
  `skills` field unless the validator requires it.
- No `dependencies` field (D4).

### 3.4 Root `README.md`

Sections, in order:

1. What this marketplace is (two plugins, one process; a short diagram of
   Cowork → HandOff → Claude Code → Report → Cowork).
2. **Connect to Cowork** — Customize → Plugins → Add marketplace →
   `https://github.com/oleksandr-ieremchuk/digiteam-cowork-marketplace`; install
   `delivery-orchestrator`.
3. **Connect to Claude Code** —
   `/plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace`,
   then `/plugin install delivery-executor@digiteam`; plus the team variant via
   `.claude/settings.json` (`extraKnownMarketplaces` + `enabledPlugins`).
4. **Prerequisite: superpowers** — both skills call
   `superpowers:brainstorming`, `superpowers:writing-plans`,
   `superpowers:executing-plans`; link to the superpowers marketplace and the
   install command. (D4)
5. **The process in one screen** — the phase table (owner / gate), copied from
   `plugins/delivery-orchestrator/skills/orchestrating-delivery/references/phase-map.md`.
6. **Repo conventions** — `design_docs/`, `engineering_plans/{drafts,ongoing,done,documented}`;
   this repo follows them itself.
7. Versioning: bump `version` in both manifests; tag `vX.Y.Z`.
8. License.

Per-plugin `README.md`: one paragraph on which tool it is for, the skill it
contains and its trigger phrases, and a pointer to the root README.

### 3.5 Git / GitHub

- `git init` in the repo folder, default branch `main`.
- Create the **public** GitHub repo `oleksandr-ieremchuk/digiteam-cowork-marketplace`
  (description: "DigiTeam plugin marketplace for Claude Cowork and Claude Code").
- Push `main`. Tag `v0.1.0` after the first push passes verification.
- Commit granularity: spec + process folders → manifests + READMEs + LICENSE →
  plugin content; or as the plan prescribes.

## 4. Behaviour of the two plugins (unchanged protocol)

- `delivery-orchestrator:orchestrating-delivery` — the existing Cowork skill,
  generic wording. Phases 0a, 1, 3, 5, 6; emits HandOffs; parses Reports.
- `delivery-executor:executing-delivery-handoff` — new. Triggers on a pasted
  `Delivery HandOff — <slug> — repo: <repo>` block. Verifies repo + plan stage
  against the requested phase, runs exactly that phase (0b / 2 / 4 / 7 / 8),
  keeps stage folders truthful with `git mv` + commit, prints the Report.
- HandOff and Report shapes are **identical** in both plugins' references
  (`handoff-prompt.md` ↔ `handoff-format.md`, `report-format.md` ↔
  `report-format.md`), including the blocked variants `Phase completed: none —
  blocked` / `Plan stage now: none`. This is the cross-plugin contract that integration
  verification (phase 5) checks.

## 5. Content already authored by Cowork (commit as-is)

Cowork has placed the following **14 files** in the working folder at their
final paths (13 under `plugins/` + this spec). They are the product; Code does
not rewrite them (typo fixes are fine, report them as deviations):

- `design_docs/marketplace-bootstrap-design.md` (this file)
- `plugins/delivery-orchestrator/skills/orchestrating-delivery/SKILL.md` + `references/*` (1 + 8 = 9 files)
- `plugins/delivery-executor/skills/executing-delivery-handoff/SKILL.md` + `references/*` (1 + 3 = 4 files)

Amended after plan review (2026-09-08): this spec and the orchestrator's
`references/report-format.md` were edited by Cowork; their hashes differ from
the plan's inventory — re-baseline those two.

Known caveat: the orchestrator skill was recovered from a synced copy in which
three files were truncated at the tail (`SKILL.md`, `references/phase-map.md`,
`references/state-discovery.md`). Cowork reconstructed the missing endings (a
few lines each). Sasha should diff against his original before v0.1.0 is tagged.

## 6. Out of scope

- Hooks, agents, MCP servers in either plugin.
- Publishing to any Anthropic-curated directory.
- CI for schema validation (nice-to-have for a later feature).
- Migrating the personal Cowork skill off the account (Sasha does that by hand
  once the marketplace version is installed, to avoid duplicate triggers).

## 7. Acceptance criteria (phase 3 / 5 gates)

1. Repo layout matches §3.1 exactly; all 14 Cowork-authored files present,
   byte-identical to the re-baselined inventory apart from reported typo fixes.
2. `.claude-plugin/marketplace.json` and both `plugin.json` are valid JSON and
   pass `claude plugin validate .` (or the current equivalent) with no errors.
3. `grep -ri "chatrevenue\|sasha" plugins/` returns nothing.
4. Every `references/*.md` file mentioned in either `SKILL.md` exists at that path.
5. Locally: `/plugin marketplace add ./` (or `claude plugin marketplace add ./`)
   succeeds, `claude plugin install delivery-executor@digiteam` succeeds, and
   the skill `executing-delivery-handoff` appears in the skill list.
6. GitHub repo is public, `main` pushed, README renders with the connect
   instructions for both tools.
7. `engineering_plans/{drafts,ongoing,done,documented}/` exist and the plan for
   `marketplace-bootstrap` sits in the stage folder matching the phase reported.

## 8. Integration verification (phase 5, run by Cowork, read-only)

- Marketplace is reachable at the public URL; `marketplace.json` parses; both
  `source` paths resolve to folders with `plugin.json` + `skills/*/SKILL.md`.
- Protocol parity: HandOff template in the orchestrator ≡ HandOff shape in the
  executor; Report shape identical in both `report-format.md` files.
- Cowork: add the marketplace from the GitHub URL, install
  `delivery-orchestrator`, confirm the skill triggers on "where are we on this
  feature".
