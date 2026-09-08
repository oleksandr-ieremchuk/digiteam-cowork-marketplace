# DigiTeam plugin marketplace — Architecture

> Living architecture. Updated as a development stage via the documentation pass
> (see the marketplace README, "Repo conventions"). Reflects the repository as
> built **today**.

## Overview

This repository is a **Claude plugin marketplace** named `digiteam`. It has no
application code: the system is a repository *shape* that two Claude products
consume. Claude Cowork and Claude Code each add the repo as a marketplace
(separately — the two products do not share plugin stores) and install the
plugin meant for them.

The marketplace ships two plugins that are two halves of one **delivery
process**: `delivery-orchestrator` runs in Cowork and owns the spec, the review
gates, cross-repo integration verification and documentation prose;
`delivery-executor` runs in Claude Code and owns everything that touches code
and git inside one repository. The halves never share state — they talk only
through two chat blocks, a **HandOff** (Cowork → Code) and a **Report** (Code →
Cowork), whose shapes are defined redundantly in both plugins and must stay
identical.

The load-bearing idea is that **process state lives on disk in the target
repo, not in either tool**: a feature's phase is read from which stage folder
(`engineering_plans/{drafts,ongoing,done,documented}`) its plan sits in. This
repository follows the same convention for its own features.

## Core building blocks

- **Marketplace manifest** — `.claude-plugin/marketplace.json`. Declares the
  marketplace name `digiteam` (the suffix users type in
  `<plugin>@digiteam`), the owner, a marketplace-level `metadata.version`, and
  one entry per plugin with a relative `source`. Both products read the same
  file. Detail: [references/packaging.md](references/packaging.md).
- **Plugins** — `plugins/<name>/` each with `.claude-plugin/plugin.json`, a
  `README.md`, and one skill under `skills/<skill>/`. Skills are auto-discovered
  from `skills/`; no `skills`, `hooks`, `agents`, `mcpServers` or
  `dependencies` fields are declared.
- **Skills** — a `SKILL.md` (frontmatter `name` + `description` that drives
  triggering, then the procedure) plus `references/*.md` the skill reads on
  demand. The orchestrator skill carries 8 references (phase map, HandOff
  template, Report format, state discovery, integration verification, the
  documentation pass, the architecture-doc template, and an optional in-repo
  README lifecycle text); the executor carries 3 (HandOff format, phase map
  from the executor's side, Report format).
- **The HandOff / Report protocol** — the only interface between the two
  plugins. Chat text, never files. Detail:
  [references/delivery-protocol.md](references/delivery-protocol.md).
- **Process folders** — `design_docs/` (specs, this architecture set) and
  `engineering_plans/{drafts,ongoing,done,documented}/`. Only Claude Code moves
  a plan between stages, always as `git mv` + commit.
- **Release surface** — public GitHub repo, `main` branch, annotated tags
  `vX.Y.Z` on the verified commit. `.gitattributes` forces LF so skill files
  stay byte-stable across Windows clones.

## Reference manifest

| Subsystem | Reference | What it covers |
|---|---|---|
| Delivery protocol | [references/delivery-protocol.md](references/delivery-protocol.md) | The HandOff and Report chat blocks, the phase/gate/stage vocabulary both plugins share, and the parity rule that keeps the two copies identical. |
| Packaging & release | [references/packaging.md](references/packaging.md) | Manifest shapes and the fields deliberately left out, how the two products install from the repo, versioning and tagging, line-ending policy. |

## Decisions

Why it's built this way lives in the append-only log: [decisions/](decisions/).
