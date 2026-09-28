# 0007 — The executor's slash command is the first line of every HandOff

- **Status:** accepted
- **Date:** 2026-09-18
- **Source:** `design_docs/executor-trigger-design.md` (D1–D4); plan `executor-trigger`

## Context
A skill is auto-invoked when the model matches its `description` against the
user's message. In the `marketplace-bootstrap` run that never happened: three
pasted HandOffs, zero invocations — two phase-7 reviews and one phase 8, the
last of which only ran because the user typed the skill's slash command by hand.
A long pasted block is evidently a weak match for a description, and a delivery
process whose executor may or may not wake up is not a process.

## Decision
- Every HandOff begins with `/delivery-executor:executing-delivery-handoff`
  alone on the first line; the block follows. A message starting with a slash
  command invokes that skill and passes the remainder as its argument.
- The skill stays **model-invocable** — no `disable-model-invocation`. The slash
  line is a guarantee on top of the plain-paste path, not a replacement for it.
- An **empty argument** is handled explicitly: the skill asks for the HandOff and
  stops, rather than guessing or scanning the repo. This is the documented
  fallback if a client ever collapses a multi-line paste.
- The older `Do:` bullet that asked Code to "run this HandOff with
  `executing-delivery-handoff` … if installed" is dropped as redundant.

## Consequences
Invocation no longer depends on description matching. When the plugin is not
installed the command is rejected outright, which is a visible failure the user
can act on instead of a silent one — and the block, still self-describing,
can be pasted without its first line as a fallback. The slash line joins the
parity rule, so both copies of the HandOff shape must carry it.

One consequence is worth naming because it was observed: the slash command fires
wherever the plugin is installed, including the orchestrator's side, so a
HandOff pasted into the wrong window does invoke the executor skill there. What
stops it is the executor's own first precondition — no working copy of the named
repo, no action. That precondition is therefore load-bearing for more than repo
mix-ups.
