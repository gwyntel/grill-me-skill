# Harness adapters

The workflow in `SKILL.md` is the same everywhere; the *surface* it rides on is not. This
file maps the round onto each harness: what to call, how wide a round can be, how the
recommended answer is expressed, whether a subagent may ask the user, and where the skill
file has to live to be seen.

Verified 2026-11-16 against primary sources (official docs, SDK types, tool schemas in
source). Tool names drift — when a name here does not exist in your tool list, believe
your tool list, fall back to the prose round, and update this file.

## At a glance

| Harness | Question tool | Questions per call | Options | Recommended option | Free text |
|---|---|---|---|---|---|
| Hermes | `clarify` | 1–5 | ≤4, objects implicit | UI labels choice 1 "(Recommended)" | automatic "Other" row |
| Claude Code | `AskUserQuestion` | 1–4 | 2–4, `label` + `description` | no field; first option is the convention | automatic "Other" |
| Codex CLI | `request_user_input` (experimental, Plan mode) | "prefer 1, do not exceed 3" | 2–3 | first option + `(Recommended)` in label | automatic (`isOther` forced) |
| Cline | `ask_question` (legacy `ask_followup_question`) | **1** | 2–5 bare strings, no multi-select | option[0] pre-selected in TUI | automatic "Type a response…" |
| OpenClaw | `ask_user` | 1–3 | 2–4, `label` + `description` | first option + `(Recommended)` in label | automatic "Other" |
| DeepSeek Harness | `ask_user_question` | 1+ (no documented cap) | optional; open-ended allowed | first option + `(Recommended)` in label | `custom` field, no auto row |
| Pi | none | — | — | prose | prose |
| Antigravity | `ask_question` | array (cap undocumented) | ≥2 bare strings, `is_multi_select` | first option + `(Recommended)` prefix | automatic "Other (write your answer)" |

Subagent rule, all of them: **the interviewer stays in the main thread.** No harness here
lets a child agent ask the user.

## Hermes

- **Tool** `clarify`. 1–5 questions per call; single-select choices are up to 4 with the
  first labelled "(Recommended)" and an "Other" free-text row appended; `multi_select` for
  scope lists; omit `choices` for an open question. Returns one response per question with
  status `answered | skipped | unanswered`.
- **Round** = at most 5 independent questions in one call. Do not write "(Recommended)" into
  a choice label — the UI adds it.
- **Subagents** — children have no `clarify`; only the main agent can interview.
  `delegate_task` is for facts.
- **Skills** — `~/.hermes/skills/<category>/<name>/SKILL.md`, discovered by `skills_list`,
  loaded with `skill_view`. Frontmatter: `name`, `description` (60-char cap), optional
  `author`, `license`, `version`, `metadata.hermes.tags`.
- **Friction** — `clarify` blocks the turn; a skipped question comes back empty, not as an
  assumed answer.

## Claude Code

- **Tool** `AskUserQuestion`: `{questions:[{question, header (≤12 chars), multiSelect,
  options:[{label, description}]}]}`, 1–4 questions, 2–4 options, "Other" always available.
  No recommended/default field — the convention is first option plus explicit wording.
- **Round** = up to 4 questions. Keep `header` short.
- **Subagents** — Anthropic's own docs: *"AskUserQuestion is not currently available in
  subagents"*. The delegation tool is `Agent` with `subagent_type` (formerly `Task`);
  subagents take `tools` lists, so they are for research, not interviewing.
- **Skills** — `.claude/skills/<name>/SKILL.md` (project) and `~/.claude/skills` (user),
  Agent Skills standard plus Claude Code extensions (`allowed-tools`,
  `disable-model-invocation`, `context: fork`, `agent`, `user-invocable`). Invoked by the
  `Skill` tool or `/name`.
- **Friction** — headless (`-p`) has no answerer; plan mode is read-only.

## Codex CLI

- **Tool** `request_user_input` (experimental; first landed as `ask_user_question`).
  `{questions:[{id, header (≤12), question, options:[{label, description}]}]}` →
  `{answers:{<id>:[labels]}}`. "Prefer 1 and do not exceed 3" questions, 2–3 mutually
  exclusive options, recommended option first with `(Recommended)` in the label; free text
  is forced on every question and returns as a `user_note`.
- **Round** = **3 questions max**, and prefer fewer. The TUI asks them one at a time anyway,
  so a "wide round" is a convenience for the model, not the user.
- **Subagents** — `spawn_agent` (root-only; `task_name` + `message`, `fork_turns`, depth 1,
  4 concurrent by default) plus `send_message`, `followup_task`, `wait_agent`,
  `interrupt_agent`, `list_agents`. **"request_user_input can only be used by the root
  thread"** — a spawned interviewer cannot ask.
- **Skills** — `.agents/skills` from CWD up to the repo root, `~/.agents/skills`, admin
  `/etc/codex/skills`; legacy `.codex/skills` still read. Frontmatter: `description`
  required, `name` optional, unknown keys ignored. Invoke with `$skill` or `/skills`.
- **Friction** — the tool is **Plan-mode only** by default (`request_user_input` is gated on
  Plan; a `default_mode_request_user_input` flag exists but is off). Plan mode is read-only,
  which suits "agree before implementing" — but if you are in Default mode the tool is
  absent, so fall back to the prose round rather than inventing multiple-choice text, which
  Default mode's prompt forbids.

## Cline

- **Tool** `ask_question` (the enum still carries legacy `ask_followup_question`):
  `{question: string, options: string[]}` — **one question per call**, 2–5 bare-string
  options, no multi-select, no descriptions, no recommended field. The CLI pre-selects
  option[0] so Enter accepts it; webview shows plain numbered buttons. A dismissed or empty
  answer throws "the user did not answer", so silence is never read as consent.
- **Round** — there is no multi-question round: ask **sequential single questions**, ordered
  by dependency. Encode the recommendation as `Use JWT (Recommended)` and put it first.
- **Subagents** — `spawn_agent {systemPrompt, task}` in CLI/Desktop (parallel, own tools);
  the VS Code extension had subagents disabled at 4.0.0. Either way the subagent cannot ask
  the user.
- **Skills** — `.cline/skills/` (recommended), `.clinerules/skills/`, `.claude/skills/`,
  `.agents/skills/`; global `~/.cline/skills`, `~/.agents/skills`. Required frontmatter
  `name` (must equal the directory) + `description`; strict Agent Plugin path rejects extra
  keys, so keep our `author`/`version`/`derived_from` fields out of a Cline-packaged copy.
  **`disable-model-invocation` is not implemented** — use `disabled: true`. Model loads it
  through `skills {skill, args}`; `/grill-me` is the reliable trigger.
- **Friction** — plan mode blocks edits but keeps `ask_question`; headless/auto-approve runs
  do not ask at all.

## OpenClaw

- **Tool** `ask_user`: `{questions:[{id (snake_case), header (≤12), question,
  options:[{label, description}]}], timeoutSeconds}` → `{status: answered|no_answer,
  answers:{answers:{<id>:[labels]}}}`. 1–3 questions, 2–4 options, no native recommended
  field (first option + `(Recommended)` by convention), automatic "Other" that you must not
  author. Default timeout 900 s, clamped 30–3600 s.
- **Round** = up to 3 questions. Only a *single* single-select question keeps native
  buttons on Telegram/Discord/Slack; wider rounds degrade to text (the web UI keeps a
  stepper). On messaging channels, prefer one question per round.
- **Subagents** — `sessions_spawn` (`context: isolated|fork`, `runTimeoutSeconds`,
  `worktree`), `sessions_yield`, `subagents`, `sessions_send`; 8 concurrent, depth 5.
  `ask_user` is **main-session only** — a spawned interviewer cannot ask.
- **Skills** — precedence `<workspace>/skills` > `<workspace>/.agents/skills` >
  `~/.agents/skills` > `<state-dir>/skills` > workshop > bundled > `skills.load.extraDirs`.
  Frontmatter: `name`, `description`, optional `user-invocable`,
  `disable-model-invocation`, `metadata.openclaw.requires`. Invoke `/skill <name>` or
  `/name`; the model can also use `skills_search` / `skills_read`.
- **Friction** — options are mandatory (no open-ended type; fake it with options + implicit
  Other); expiry returns `no_answer` and the agent proceeds.

## DeepSeek Harness (`dsh`)

- **Tool** `ask_user_question`: `{questions:[{id, question, header?, options:[{label,
  description?}], multi_select?}]}` → `{answers:[{id, selected:[labels], custom?}]}`. Ids
  are echoed; options are optional, so open-ended questions work. No documented per-call
  cap. Blocking by default; an optional `mode: timed` lets a deadline release the model
  while the question stays answerable. Their schema text is explicit: *"If you recommend
  one, put it first and append `(Recommended)` to that label."*
- **Round** — as many independent questions as the frontier needs; ids make the answers
  unambiguous. Give every question a distinct snake_case id.
- **Subagents** — a first-class subsystem (`tool-subagent-control`, depth limits, parent or
  caller delivery) — but **"a live child agent cannot call the tool"**, so no delegated
  interviewer.
- **Skills** — directory bundles `<name>/SKILL.md` or flat `<name>.md`, scanned in rank
  order: `.dsh/skills` (100) → `.agents/skills` (200) → `Config.customSkillDirs` (300) →
  `<dshHome>/skills` (400) → `<agentsHome>/skills` (500) → bundled (600). Frontmatter reads
  kebab-case `disable-model-invocation` and `user-invocable`, both default `true`. **No
  nested `**/SKILL.md` recursion.** Invoked through the `skill` tool. Developer preview:
  breaking changes expected.

## Pi

- **Tool** — **none.** Pi's model-facing toolset has no ask-user tool; prompt surfaces exist
  only inside TypeScript extensions (`ctx.ui.select / confirm / input / editor`), which the
  model cannot call.
- **Round** — always the **prose round**: number the questions, recommendation under each,
  then stop. This is exactly upstream's original format.
- **Subagents** — none built in; an *example* extension (`examples/subagent/`) registers one.
  Treat as absent.
- **Skills** — full Agent Skills spec: a directory with `SKILL.md`, discovered recursively;
  project and user skills dirs plus `~/.agents/skills/` and `.agents/skills/` (walked up to
  the repo root). `disable-model-invocation: true` honoured; force with `/skill:name`, and
  text after the name is appended as a user request.
- **Friction** — describe the skill as both *what* and *when*, or Pi will not route to it.

## Antigravity

- **Tool** `ask_question` (SDK `BuiltinTools.ASK_QUESTION`): `{questions:[{question,
  options: string[], is_multi_select}], toolSummary, toolAction}`. ≥2 plain-string options;
  the card adds numbering and an "Other (write your answer)" field; no recommended field —
  prefix option 1 with `(Recommended)`, and strip the prefix from the returned string. Every
  tool also needs the `toolSummary` / `toolAction` strings.
- **Round** — the array allows several questions, but the shipped UX is **one card at a
  time** and the platform prompt steers hard toward one question at a time. Prefer single
  questions; do not fight it.
- **Subagents** — `invoke_subagent` (`TypeName`, `Role`, `Prompt`, `Workspace:
  inherit|branch|share`), `define_subagent`, `send_message`, `manage_subagents`; depth 1.
  `ASK_QUESTION` needs an interactive agent, so a subagent is not an interviewer.
- **Skills** — `.agents/skills/<name>/SKILL.md` (workspace, parent dirs scanned), global
  `~/.gemini/config/skills/`, CLI extras `~/.gemini/antigravity-cli/skills/` and
  `~/.gemini/skills/`. Frontmatter documented as `name` + `description` only; plugins
  namespace as `<plugin>:<skill>`; invocation `/skill-name` or autonomous.
- **Friction** — **Antigravity ships a native `/grill-me`** (one question at a time). Do not
  duplicate its UX: prefer one-question rounds there, and say so once instead of batching.
  Headless `agy -p` has no answerer (run ends `WAITING`).

## Unknowns (do not guess these)

- Codex: whether `request_user_input` has shipped in the latest tagged release — public Codex
  docs never mention it, so a released binary may not have it. Fall back to prose.
- Cline: whether several parallel `ask_question` calls in one turn render as a batch (the
  shared UI component takes `items[]`, the tool schema does not).
- Antigravity: per-call question and option caps; subagent concurrency.
- DeepSeek `dsh`: per-call question cap (undocumented; the schema is unbounded).
- Claude Code: whether a future release enables `AskUserQuestion` inside subagents. Today's
  docs say no.
