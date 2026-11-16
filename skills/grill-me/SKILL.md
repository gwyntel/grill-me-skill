---
name: grill-me
description: Grill a plan or design in rounds of questions.
version: 1.1.0
author: gwyntel
license: MIT
derived_from: mattpocock/skills skills/productivity/grilling/SKILL.md @ 5863806dd4e15c63174bee4e7515a4256792ecfa
metadata:
  hermes:
    tags: [planning, clarify, design, interview, requirements, portable]
    related_skills: [development-workflows, ideation, code-review]
---

# Grill Me — Round-by-Round Design Interview

Interview the user relentlessly until you reach a shared understanding, then stop. The
output of a grilling session is agreement, not code.

Use this when the user says "grill me", "grill this plan", "stress-test this", "poke holes
in it", "interview me about this", or when a task is genuinely multi-decision (new project,
refactor, migration, anything where you would otherwise guess at three or more unsettled
choices). Skip it for single-answer questions, for work already specified, and for work
already under way with an agreed design.

This skill is harness-neutral. `references/harness-adapters.md` carries the verified
question tool, per-call cap, subagent rule, skill-install path and frontmatter quirks for
Hermes, Claude Code, Codex, Cline, OpenClaw, DeepSeek Harness, Pi and Antigravity. Read it
when you are not on Hermes, or when the surface behaves differently than described below.

## The design tree and the frontier

Map the work as a **design tree**: every decision branches into the decisions hanging off
it. The **frontier** is every decision whose prerequisites are already settled — the
questions you can ask *now* without guessing at answers you have not heard.

You never ask the whole tree. You ask the frontier, one round at a time. Each round of
answers pushes the frontier outward and opens questions that were blocked.

## A round is one call to the question tool

Every harness with a structured-question surface caps how much one call can carry — one
question (Cline), three (Codex, OpenClaw), four (Claude Code), five (Hermes), or
undocumented (DeepSeek Harness, Antigravity). **Probe the cap once, then treat it as the
round size.** You know what the harness offers from the tool description in front of you;
do not assume the Hermes number elsewhere, and do not spend a round asking about its own
limit.

- **Ask the whole frontier in one call**, up to the cap. Independent questions go in a
  single call — never a chain of one-question calls when a wider call would fit.
- **Frontier larger than the cap:** order by fan-out and ask the widest set that fits. The
  rest wait; their answers may have changed by then anyway.
- **Independent questions only.** If Q2's answer depends on Q1 in the same call, Q2 belongs
  to a *later* round. Dependent questions asked together produce answers that contradict
  each other.
- **No such tool, or no answerer** (headless runs, Plan-mode-only tools, `codex exec`,
  `agy -p`, auto-approve): fall back to one **prose round** — number the questions, put
  the recommendation directly under each one, then stop and wait. That is the upstream
  format, and it is the correct degraded form, not a failure.
- **Never enumerate options in question prose** when the harness renders choices. Options
  belong in the tool's options field; prose options are dead text the user cannot click.

## Where the interview runs

**In the main thread, always.** On nearly every harness a child agent cannot reach the
user: Codex answers `request_user_input` only on the root thread, OpenClaw restricts
`ask_user` to the main session, a live DeepSeek child cannot call its tool at all, Claude
Code states `AskUserQuestion` is unavailable in subagents, and Hermes children have no
`clarify`. Delegating the interviewer strands the session — so gather facts with subagents
if you like, but ask the user yourself.

## Shaping each question

- **Recommended option first.** That is the portable contract: Cline pre-selects the first
  option so Enter accepts it, and Codex, OpenClaw, DeepSeek and Antigravity all document
  "put the recommended option first and suffix its label `(Recommended)`". Hermes and
  Claude Code label the first choice themselves — **do not duplicate the marker there**;
  add it only where the harness has no native marking.
- **Respect the option count.** Two to four is the safe band everywhere; Codex wants two to
  three, Cline rejects fewer than two or more than five, Claude Code and Antigravity accept
  two to four. If the real decision space is not enumerable into honest options, ask
  open-ended (Hermes, DeepSeek and Antigravity allow no options at all).
- **Never author an "Other".** Every surface with options appends its own free-text row.
  A hand-written "Other" duplicates it and reads as noise.
- **Multi-select** only for "which of these apply" — scope lists, features, acceptance
  criteria. Mutually exclusive forks are single-select.
- **Each option must be a decision**, not a restatement of the question — "Postgres with a
  migration script" beats "yes, use a database".
- **Title each question by the decision it settles** ("Retention policy", not "Question
  3"), and keep headers short: Codex and OpenClaw truncate at 12 characters.
- **Word it so a one-word answer is usable.** If "yes" would be ambiguous, the question is
  not finished.

## Facts are yours. Decisions are theirs.

Finding facts is your job, never the user's. Anything from the environment — files, config,
git history, a dependency's real API, how a repo is laid out — you look up yourself, or
send to a subagent when it is a fat search (fresh context, not a fork of this one).

- Do not spend a question on something you could have checked.
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

The answers are the only record of what was decided, and sessions compact.

- Open each round with one line of **settled** decisions ("Settled: Postgres, single-tenant,
  cutover in one migration").
- For anything longer than three rounds, keep the tree in a scratch file (`grill-<topic>.md`
  in the session's scratch or working directory) with decided / open / blocked nodes, and
  rewrite it each round.
- A question the user skipped or left unanswered stays **open**. Every harness reports a
  skip distinctly — an empty selection, `no_answer`, a thrown "user did not answer". Never
  infer a decision from silence, and never re-ask it identically; reframe or retire the
  branch.
- Antigravity and Cline echo the `(Recommended)` marker inside the answer string; strip it
  before recording the decision.

## Closing

The session is done when the frontier is empty: every branch visited, nothing silently
assumed.

1. State the shared understanding back as a decision list, compactly.
2. Ask once whether to proceed or revise (proceed first, recommended; one question, not a
   round).
3. **Do not implement, write files, or dispatch build work before that confirmation.** If
   the user says "just do it", the tree collapses to what is decided — implement exactly
   that and name every assumption you are carrying.

If the user asks for one question at a time, honour it: single-question calls. The
round-based default is a default, not a rule — and on Antigravity and Cline the platform
already behaves that way, so do not fight it.

## Pitfalls

- **Twelve questions in prose** ("a few quick questions…") — the exact failure the question
  tool exists to prevent. Escaped questions get partial answers.
- **Dependent questions in one call** — the answers arrive contradictory and you cannot
  tell which one wins.
- **Assuming the cap.** A five-question round on Codex or Cline is a rejected or truncated
  call. Probe, then size the round to the surface.
- **Delegating the interviewer** to a subagent that cannot ask the user.
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
router that invokes `grilling`) from the workflow; this port keeps one skill because not
every harness has a way to invoke a sibling skill by name. Adapted for this runtime:
rounds are structured-question calls sized to the surface's cap, the "recommended answer"
paragraph becomes the first option, sub-agents become the host's delegation tool, and
state persistence plus the confirmation gate are additions. The harness adapter table is
this port's own research, not upstream's. See `NOTICE.md`.
