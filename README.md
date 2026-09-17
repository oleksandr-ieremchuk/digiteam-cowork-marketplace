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

Every Delivery HandOff Cowork produces begins with the line
`/delivery-executor:executing-delivery-handoff`, so pasting the whole block
into Claude Code as a single message runs the executor on it directly. If
Claude Code reports that command as unknown, the plugin is not installed — run
the two commands above.

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

Both skills delegate the thinking-heavy steps to the `superpowers` skills, but
not to the same ones. The orchestrator uses `superpowers:brainstorming`
(phase 0a), `superpowers:writing-plans` (0b) and `superpowers:executing-plans`
(2); the executor uses `superpowers:writing-plans` and
`superpowers:executing-plans`. Install that marketplace too:

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

`v0.1.1` is the first bump after the initial release. Both plugins move
together even when only one of them changed, so all five `version`
occurrences change on every release.

## License

MIT — see [LICENSE](LICENSE).
