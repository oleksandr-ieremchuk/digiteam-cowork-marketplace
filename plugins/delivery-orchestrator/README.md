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
