# grill-me-skill

Round-by-round design interview for coding agents — one portable workflow, adapted to the
question surface each harness actually has.

"Grill me about this plan" turns into a **design tree** walked in **rounds**: each round
carries the whole *frontier* — the decisions whose prerequisites are already settled — with
the recommended answer already first, so the user can accept it with one click. Answers push
the frontier outward. The session ends when the frontier is empty and the user confirms
shared understanding. No code gets written before the gate.

Ported from Matt Pocock's `grilling` skill (`mattpocock/skills`, MIT) — see `NOTICE.md`.

## Harnesses covered

| Harness | Question surface | Round width |
|---|---|---|
| Hermes | `clarify` | 5 |
| Claude Code | `AskUserQuestion` | 4 |
| Codex CLI | `request_user_input` (experimental, Plan mode) | 3 |
| Cline | `ask_question` | 1 (sequential) |
| OpenClaw | `ask_user` | 3 |
| DeepSeek Harness | `ask_user_question` | unbounded |
| Pi | none — prose round | — |
| Antigravity | `ask_question` (native `/grill-me` already ships) | 1 preferred |

Details, exact parameter shapes, subagent rules and install paths per harness:
`skills/grill-me/references/harness-adapters.md`.

## Why it exists

Asking twelve questions as prose gets partial answers. Asking one at a time gets slow
answers. A frontier round is the middle: only questions that cannot contradict each other,
all of them at once, each one pre-answered so the user can accept the default with a click.

Two rules travel everywhere: **the interviewer stays in the main thread** (no harness here
lets a subagent ask the user), and **the recommended option goes first** (Cline pre-selects
it; Codex, OpenClaw, DeepSeek and Antigravity document `(Recommended)` in the label; Hermes
and Claude Code mark it themselves).

## Layout

```
skills/grill-me/SKILL.md                      # the workflow, harness-neutral
skills/grill-me/references/harness-adapters.md # per-harness surface map (verified 2026-11-16)
LICENSE.txt                                   # upstream MIT licence, unmodified
NOTICE.md                                     # upstream, revision, what was adapted
```

## Install

Vercel `skills` CLI:

```sh
npx skills@latest add gwyntel/grill-me-skill --skill=grill-me
```

Copy the directory to the harness's skill root — every path below is a directory containing
`SKILL.md`:

| Harness | Install to |
|---|---|
| Hermes | `~/.hermes/skills/productivity/grill-me` |
| Claude Code | `~/.claude/skills/grill-me` or `<repo>/.claude/skills/grill-me` |
| Codex CLI | `~/.agents/skills/grill-me` or `<repo>/.agents/skills/grill-me` |
| Cline | `~/.cline/skills/grill-me` or `<repo>/.cline/skills/grill-me` |
| OpenClaw | `~/.agents/skills/grill-me` or `<workspace>/skills/grill-me` |
| DeepSeek Harness | `<dshHome>/skills/grill-me` or `<repo>/.agents/skills/grill-me` |
| Pi | `~/.agents/skills/grill-me` or `<repo>/.agents/skills/grill-me` |
| Antigravity | `~/.gemini/config/skills/grill-me` or `<repo>/.agents/skills/grill-me` |

```sh
cp -R skills/grill-me ~/.hermes/skills/productivity/grill-me
# never git clone into a skills tree that is itself a git repo — copy, then rm -rf .git
```

Confirm discovery (`skills_list` on Hermes, `/skills` on Codex, `/skill:grill-me` on Pi).

## Use

Load it before a genuinely multi-decision task: new project, refactor, migration, or any
request where you would otherwise guess at three or more unsettled choices. Or say "grill
me" and let the trigger phrase fire.

Requirements: a way to ask the user structured questions (see the table), plus whatever the
host uses for subagent fact-finding so no question slot is spent on something the agent
could look up.

## Update

```sh
npx skills update grill-me
```

Then mirror `SKILL.md` and `references/` into the installed copy — the repo and the install
are separate, and the install is what the agent actually loads.

## Troubleshooting

- **Skill not listed after install** — frontmatter must keep `name` and `description`; if the
  host caps the description, it is 60 characters on Hermes. Cline requires `name` to equal
  the directory name.
- **A harness rejects the frontmatter** — Cline's strict Agent Plugin packaging rejects extra
  keys (`author`, `version`, `derived_from`); strip them for a Cline-packaged copy.
- **Rounds feel like an interrogation** — the frontier is being computed as "all questions"
  rather than "questions with settled prerequisites". Split coarser branches first.
- **Answers contradict each other** — dependent questions were asked in one call. Move the
  dependent one to the next round.
- **A question call errors or is truncated** — the round was wider than the harness's cap.
  Check the table, halve the round.
- **The tool does not exist** (Codex Default mode, headless runs, Pi) — use the prose round:
  number the questions, recommendation under each, then wait.
- **User wants one question at a time** — supported; the round default is a default, not a
  rule. Antigravity and Cline behave that way natively; do not fight them.
