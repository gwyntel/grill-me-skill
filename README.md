# grill-me-skill

Round-by-round design interview for Hermes, driven by the `clarify` tool.

"Grill me about this plan" turns into a **design tree** walked in **rounds**: each round is
one `clarify` call carrying the whole *frontier* — the decisions whose prerequisites are
already settled — each question with its recommended answer already picked. Answers push
the frontier outward. The session ends when the frontier is empty and the user confirms
shared understanding. No code gets written before the gate.

Ported from Matt Pocock's `grilling` skill (`mattpocock/skills`, MIT) — see `NOTICE.md`.

## Why it exists

Asking twelve questions as prose gets partial answers. Asking one at a time gets slow
answers. A frontier round is the middle: only questions that cannot contradict each other,
all of them at once, each one pre-answered so the user can accept the default with a click.

## Layout

```
skills/grill-me/SKILL.md   # the whole workflow
LICENSE.txt                # upstream MIT licence, unmodified
NOTICE.md                  # upstream, revision, what was adapted, what was not copied
```

## Install

Vercel `skills` CLI:

```sh
npx skills@latest add gwyntel/grill-me-skill --skill=grill-me
```

Hermes (installed copy is a plain directory, not a symlink into the repo):

```sh
cp -R skills/grill-me ~/.hermes/skills/productivity/grill-me
# never git clone into ~/.hermes/skills — that tree is itself a git repo
```

Confirm discovery with `skills_list`; the skill loads as `grill-me` (or
`productivity/grill-me` if the name collides with something else installed).

## Use

Load it before a genuinely multi-decision task: new project, refactor, migration, or any
request where you would otherwise guess at three or more unsettled choices. Or say "grill
me" and let the trigger phrase fire.

Requirements: the `clarify` tool (5 questions per call, 4 choices each, recommended option
first) and, optionally, `delegate_task` for fact-finding so no clarify slot is spent on
something the agent could look up.

## Update

```sh
npx skills update grill-me
```

Then mirror `SKILL.md` into the installed copy — the repo and the install are separate, and
the install is what the agent actually loads.

## Troubleshooting

- **Skill not listed after install** — frontmatter must keep `name` and `description`; the
  description is capped at 60 characters and the index clips the rest.
- **Rounds feel like an interrogation** — the frontier is being computed as "all questions"
  rather than "questions with settled prerequisites". Split coarser branches first.
- **Answers contradict each other** — dependent questions were asked in one call. Move the
  dependent one to the next round.
- **User wants one question at a time** — supported; the round default is a default, not a
  rule. Say so once and use single-question `clarify` calls.
