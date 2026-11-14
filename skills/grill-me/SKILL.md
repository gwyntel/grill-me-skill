---
name: grill-me
description: Interview a plan or design in rounds of clarify questions.
version: 1.0.0
author: gwyntel
license: MIT
derived_from: mattpocock/skills skills/productivity/grilling/SKILL.md @ 5863806dd4e15c63174bee4e7515a4256792ecfa
metadata:
  hermes:
    tags: [planning, clarify, design, interview, requirements]
    related_skills: [development-workflows, ideation, code-review]
---

# Grill Me — Round-by-Round Design Interview

Interview the user relentlessly until you reach a shared understanding, then stop. The
output of a grilling session is agreement, not code.

Use this when the user says "grill me", "grill this plan", "stress-test this", "poke holes
in it", "interview me about this", or when a task is genuinely multi-decision (new
project, refactor, migration, anything where you would otherwise guess at three or more
unsettled choices). Skip it for single-answer questions, for work already specified, and
for work already under way with an agreed design.

## The design tree and the frontier

Map the work as a **design tree**: every decision branches into the decisions hanging off
it. The **frontier** is every decision whose prerequisites are already settled — the
questions you can ask *now* without guessing at answers you have not heard.

You never ask the whole tree. You ask the frontier, one round at a time. Each round of
answers pushes the frontier outward and opens questions that were blocked.

## A round is a `clarify` call

`clarify` carries **1–5 questions per call** and blocks until the user submits. That cap
is the round size, so:

- **A round is at most five questions.** Ask them in one `clarify` call — never a chain of
  one-question calls when the questions are independent.
- **Frontier larger than five:** order by fan-out and ask the five that unblock the most
  downstream questions. The rest wait for the next round; their answers may have changed
  by then anyway.
- **Independent questions only.** If Q2's answer depends on Q1 in the same call, Q2 belongs
  to a *later* round. Dependent questions asked together produce answers that contradict
  each other.
- **Never enumerate options in the question text.** Options go in `choices`, where they
  render as pickable rows; prose options are dead text the user cannot click.

## Shaping each question

- **Recommended option first.** `clarify` labels the first single-select choice
  "(Recommended)" and appends a free-text row. The recommendation is a *choice*, not a
  paragraph of advice.
- **Up to four choices.** If the real decision space is not enumerable into four honest
  options, omit `choices` and ask open-ended. A padded choice list is worse than a blank.
- **Multi-select** for "which of these apply" (scope lists, acceptance criteria, things to
  keep). Single-select for mutually exclusive forks.
- **Each option must be a decision**, not a restatement of the question — "Postgres with a
  migration script" beats "yes, use a database".
- **Title each question by the decision it settles** ("Retention policy", not "Question 3"),
  so the answered form reads back as a decision log.
- **Word it so a one-word answer is usable.** If "yes" would be ambiguous, the question is
  not finished.

## Facts are yours. Decisions are theirs.

Finding facts is your job, never the user's. Anything from the environment — files, config,
git history, a dependency's actual API, how a repo is laid out — you look up yourself, or
send to `delegate_task` when it is a fat search (fresh context, not a fork of this one).

- Do not spend a clarify slot on something you could have checked.
- Do not block the round on a lookup in flight: a running exploration is an unsettled
  prerequisite, so only the questions downstream of it wait. Ask the rest of the frontier
  now.
- Batch cheap lookups *before* the round. A round asked on wrong facts burns the user's
  answers, which is the expensive resource.

The *decisions* are the user's. Put each one to them and wait.

## Recompute every round

Answers reshape the tree. Settle what is settled, retire branches the answers killed,
surface the questions the answers just made askable. Recompute the frontier from scratch
each round instead of walking a plan you wrote before the first answer.

## Keep the state somewhere durable

`clarify` answers are the only record of what was decided, and sessions compact.

- Open each round with one line of **settled** decisions ("Settled: Postgres, single-tenant,
  cutover in one migration").
- For anything longer than three rounds, keep the tree in a scratch file
  (`~/.hermes/cache/scratch/grill-<topic>.md`) with decided / open / blocked nodes, and
  rewrite it each round.
- A question the user skipped or left unanswered stays **open**. Never infer a decision
  from silence, and never re-ask it identically — reframe or retire the branch.

## Closing

The session is done when the frontier is empty: every branch visited, nothing silently
assumed.

1. State the shared understanding back as a decision list, compactly.
2. Ask once, with `clarify`, whether to proceed or revise (proceed first, recommended).
3. **Do not implement, write files, or dispatch build work before that confirmation.** If
   the user says "just do it", the tree collapses to what is decided — implement exactly
   that and name every assumption you are carrying.

If the user asks for one question at a time, honour it: single-question `clarify` calls.
The round-based default is a default, not a rule.

## Pitfalls

- **Twelve questions in prose** ("a few quick questions…") — the exact failure `clarify`
  exists to prevent. Escaped questions get partial answers.
- **Dependent questions in one call** — the answers arrive contradictory and you cannot
  tell which one wins.
- **Interrogating the user for facts** you could look up in ten seconds.
- **Asking decisions nobody cares about** — for genuinely low-stakes forks, decide it,
  say so under *settled*, and spend the round on the real forks.
- **Questions too big to answer in a form** — if it needs an essay, the branch is too
  coarse; split it.
- **Rigged choices** — stacking the recommended option with three strawmen teaches the user
  to ignore your choices. Offer real alternatives.
- **Grilling an agreed design** — once the shape is settled, more rounds are churn.

## Provenance

Ported from Matt Pocock's `grilling` skill in `mattpocock/skills` (MIT), revision
`5863806dd4e15c63174bee4e7515a4256792ecfa`. Upstream splits the entry point (`grill-me`, a
router that invokes `grilling`) from the workflow; Hermes has no Skill tool, so the pair is
merged. Adapted for this runtime: rounds are `clarify` calls capped at five questions, the
"recommended answer" paragraph becomes the first choice, sub-agents become `delegate_task`,
and state persistence plus the confirmation gate are additions. Not copied: upstream's
emoji round format, its `CLAUDE.md` opt-out line, and its docs pages. See `NOTICE.md`.
