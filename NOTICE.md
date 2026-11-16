# NOTICE

This skill is a **port** — an adaptation of someone else's workflow for this runtime — not
an original work, and not a verbatim vendored copy.

## Upstream

- Project: **Matt Pocock's skills** — "Skills for Real Engineers. Straight from my .agents
  directory." <https://github.com/mattpocock/skills> (homepage <https://aihero.dev/skills>)
- Repository: `mattpocock/skills`
- Files read: `skills/productivity/grilling/SKILL.md` (blob
  `d2f09a18e6a6c65c55f1271ba08dfeaf06f0bed3`), `skills/productivity/grill-me/SKILL.md` (blob
  `3947ff9c4ad980d14fc07fccbf659d47c114e81d`), `docs/productivity/grilling.md`,
  `docs/productivity/grill-me.md`
- Revision: `5863806dd4e15c63174bee4e7515a4256792ecfa`
- Licence: MIT (see `LICENSE.txt`, upstream text unmodified, Copyright (c) 2026 Matt Pocock)

Upstream lineage worth knowing: round-based questioning shipped briefly as the experimental
skill `in-progress/batch-grill-me`, then moved into `grilling` itself, so `grill-me` is now
a stub that invokes `grilling`. There is no `batch-grill-me` to install upstream, and any
copy of it you find (e.g. `mcowger/plexus`'s vendored `.agents/skills/batch-grill-me/`) is a
frozen July 2026 snapshot that its own refresh workflow reports as deleted upstream.

## What was adapted

The idea — a design tree walked in rounds over the frontier, facts found by the agent and
decisions put to the user — is upstream's, and the wording of several passages follows it
closely. The runtime translation is this repository's:

- A round is one `clarify` call: **1–5 questions**, up to four choices each, the recommended
  option first because `clarify` labels the first choice "(Recommended)" and appends a
  free-text row. Upstream states the recommendation as a paragraph under a numbered
  question; Hermes has no equivalent surface, so the recommendation becomes a choice.
- Frontier larger than five questions: ordered by fan-out across rounds. Upstream assumes an
  unbounded round.
- Sub-agents → `delegate_task` with a fresh context window.
- Added: state persistence across rounds (settled/decisions restated each round, optional
  scratch file), the "a skipped answer stays open" rule, and a mandatory confirmation gate
  before implementation.
- Merged: upstream's `grill-me` router and `grilling` workflow are one skill here, because
  Hermes has no Skill tool to invoke another skill by name.

## What was not copied

Upstream's emoji round-rendering template, its `CLAUDE.md` one-question-at-a-time opt-out
line (kept as prose guidance instead), its `docs/` pages, its `agents/openai.yaml`
interface metadata, and its sibling skills (`grill-with-docs`, `triage`, `wayfinder`,
`loop-me`, the `writing-*` family).

## Harness adapters

`skills/grill-me/references/harness-adapters.md` is this port's own research, not
upstream's: tool names, per-call caps, subagent rules, skill-install paths and frontmatter
quirks for eight harnesses, verified 2026-11-16 against primary sources (official docs, SDK
types, tool schemas in source). Harness and product names belong to their owners; the table
asserts facts about their public interfaces, no affiliation.

## Licence of this repository

The adapted text is distributed under the upstream MIT terms (carried forward, not
re-licensed). Upstream copyright is Matt Pocock's; the port's structure and additions are
GwynTel's.
