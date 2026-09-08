# Packaging & release

## Responsibility

How the repository presents itself to Claude Cowork and Claude Code as an
installable marketplace, how versions are declared and released, and the
repository-level policies that keep the shipped skill files stable. It does not
own the skills' content — that belongs to the process the skills describe.

## Structure

```
.claude-plugin/marketplace.json        name digiteam · owner · metadata.version · plugins[]
plugins/<name>/
  .claude-plugin/plugin.json           name · version · description · author · repository · license · keywords
  README.md                            which tool, which skill, pointer to root README
  skills/<skill>/SKILL.md + references/
README.md · LICENSE (MIT) · .gitattributes
```

## Contracts

**`marketplace.json`.** `name: "digiteam"`; `owner` (name + email);
`metadata.description`, `metadata.version`; `plugins[]` with `name`,
relative `source` (`./plugins/<name>`), `description`, `version`, `category`,
`keywords`. A `$schema` key is **rejected** by `claude plugin validate` and is
not present. The plugin `name`, `source` and `description` are the contract
users and the READMEs rely on; other fields may change.

**`plugin.json`.** `name` equal to the marketplace entry; `version` equal to the
entry's; `description` the same text as the entry; `author: { "name" }` with no
email; `repository`, `license: "MIT"`, `keywords`. No `skills` field (default
`skills/` discovery), no `dependencies`.

**Installation.** Cowork: Customize → Plugins → Add marketplace → the GitHub
URL (public repos only) → install `delivery-orchestrator`. Claude Code:
`/plugin marketplace add oleksandr-ieremchuk/digiteam-cowork-marketplace` →
`/plugin install delivery-executor@digiteam`, or per-repo via
`.claude/settings.json` (`extraKnownMarketplaces` + `enabledPlugins`). The two
products keep separate plugin stores and update differently: Claude Code has
`claude plugin marketplace update digiteam` and `claude plugin update <plugin>`
(then `/reload-plugins`), and auto-update can be switched on per marketplace in
`/plugin` (off by default for non-Anthropic marketplaces); Cowork has no update
command — remove and re-add the plugin (or the marketplace) to pick up a new
version.

**Prerequisite.** Both skills delegate to `superpowers`: the orchestrator calls
`superpowers:brainstorming`, `superpowers:writing-plans` and
`superpowers:executing-plans` (brainstorming itself in phase 0a, the other two
named in its HandOffs); the executor calls
`superpowers:writing-plans` and `superpowers:executing-plans`. This is
documented in the README, not declared as a dependency.

## Lifecycle / flow

1. Change skill content or manifests on `main` through the delivery process
   (spec → plan → execute → deploy).
2. Bump `version` in every changed `plugin.json` and the matching
   `marketplace.json` entry, and `metadata.version`; all three currently agree.
3. Deploy = push `main`, verify the remote install round trip, then create an
   annotated tag `vX.Y.Z` on the verified commit and push it. The plan's
   `ongoing → done` move follows the tag, so a tag never contains its own
   plan's `done` move.
4. Consumers pick up the new version: Claude Code via `claude plugin update`
   (or marketplace auto-update), Cowork by removing and re-adding the plugin.

## Constraints & decisions

- Public repository — [ADR 0001](../decisions/0001-public-repository.md).
- superpowers documented, not declared — [ADR 0003](../decisions/0003-superpowers-as-documented-prerequisite.md).
- Naming and versioning: `digiteam`, one shared version, plain `vX.Y.Z` tag,
  no email in `plugin.json` — [ADR 0005](../decisions/0005-naming-versioning-and-author-fields.md).
- LF line endings enforced by `.gitattributes` — [ADR 0006](../decisions/0006-lf-line-endings.md).
