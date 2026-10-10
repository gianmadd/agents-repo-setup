# Default configuration

The defaults behind `agents-setup`'s questions; the user keeps, changes or drops each one. The fenced blocks are the text written to the rules block (or to `AGENTS.local.md` where marked); `<angle brackets>` are filled from the answers. Fork this file to change the defaults.

## Pointer

Always written to `AGENTS.md`, inside the markers, so every agent also loads `AGENTS.local.md`: Claude Code through the `@` import, other agents by following the sentence.

```md
## Personal instructions

Before your first reply or action, read `AGENTS.local.md` next to this file, if it exists, and follow it as part of these instructions: @AGENTS.local.md
It holds this checkout's personal rules; it is not committed and not every agent loads it on its own. In a worktree, read it from the main checkout root, `$(git rev-parse --path-format=absolute --git-common-dir)/..`.
```

## Header

Always written, first in the rules block.

```md
# Agent instructions

Standing rules for every agent working in this repo. Personal values (GitHub account, Azure identity) are in `AGENTS.local.md`.
```

## Skills and plugins

Each entry: prerequisites, install command per agent, how to switch it off for this repo, and the text it adds.

### Feature workflow (mattpocock/skills)

- Default: on.
- Prerequisites: skills `setup-matt-pocock-skills`, `grill-with-docs`, `grilling`, `domain-modeling`, `to-spec`, `to-tickets`, `implement-spec`, `tdd`, `code-review`.
- Install (every agent): `npx skills add mattpocock/skills -g -y -s <missing skills, space-separated>`.

```md
## Workflow

New features go through the mattpocock/skills pipeline, one step at a time. The user starts each step: when one finishes, suggest the next instead of running it yourself.

1. `grill-with-docs`: interview the user until the design is settled; updates `GLOSSARY.md` and ADRs.
2. `to-spec`: turn the settled design into a spec on the issue tracker.
3. `to-tickets`: split the spec into tickets with blocking relations.
4. `implement-spec`: implement the tickets with subagents, TDD and a final code review.

If `docs/agents/` is missing, the issue tracker is not configured yet: suggest `setup-matt-pocock-skills` first.
```

### Skill nudge

- Default: on, only together with the feature workflow.

```md
When the user asks for a non-trivial task (new feature, multi-file change, refactor) outside the pipeline, ask once whether to go through it instead. Small fixes and questions go straight ahead.
```

### Ponytail (DietrichGebert/ponytail)

- Default: on.
- Prerequisite: Claude Code plugin `ponytail@ponytail` (`claude plugin list`); Codex plugin `ponytail@ponytail` (`[plugins."ponytail@ponytail"]` in `~/.codex/config.toml`); other agents, skill `ponytail` (`npx skills ls -g`).
- Install: Claude Code `claude plugin marketplace add DietrichGebert/ponytail && claude plugin install ponytail@ponytail`; Codex `codex plugin marketplace add DietrichGebert/ponytail && codex plugin add ponytail@ponytail`; other agents `npx skills add DietrichGebert/ponytail -g -y`.
- Switch off for this repo, when dropped but installed: Claude Code, `"enabledPlugins": {"ponytail@ponytail": false}` in `.claude/settings.local.json`; Codex, `[plugins."ponytail@ponytail"]` with `enabled = false` in `.codex/config.toml` (read only once the project is trusted in Codex).

```md
## Code

Write and review code the ponytail way: the plugin loads it every session; use `ponytail-review` for reviews.
```

### Added by the user

Any installed skill or plugin the user adds gets one line under `## Workflow`: when to use it, by name. If the user wants to start it themselves, write "suggest `<name>`"; otherwise "use `<name>`".

## Rules

### Subagents

- Default, per agent in use: Claude Code `sonnet`; Codex, ask for the model (show the main model from `~/.codex/config.toml` as reference). Effort `high` for implementation and review, `medium` for the rest.

```md
## Subagents

Subagents run on a cheaper model than the main session: pass model and effort explicitly at every spawn.

- Claude Code: `model: sonnet`; effort `high` for implementation and review, `medium` for research and exploration.
- Codex: model `<model>`; reasoning effort `high` for implementation and review, `medium` for research and exploration.
```

### Git

- Default: permission level (b). The GitHub lines only when the remote is on GitHub, with the account chosen by the user.
- Permission levels: (a) read-only: anything that writes to git or GitHub needs confirmation; (b) branches and local commits allowed, push, PRs, merge to main and issue changes need confirmation; (c) everything allowed except pushing to main.

```md
## Git

- <permission level as two lines: "Allowed without asking: …" and "Ask first: …">
```

GitHub remote only, appended to the section:

```md
This machine's active GitHub account, which may not be this repo's, is used by `gh`<" and by git through its `gh` credential helper">. Prefix every `gh` command<", and every `git` command that reaches GitHub (push, pull, fetch),"> with `GH_TOKEN=$(gh auth token -u <account>)`, `<account>` from `AGENTS.local.md`; if it is missing, ask the user.
```

The quoted parts only when the remote is HTTPS and git's credential helper is `gh auth git-credential`: `GH_TOKEN` reaches git only through that helper.

`AGENTS.local.md`, GitHub remote only:

```md
- GitHub account: `<account>`
```

### Azure

- Default: off; written only when the user says the repo uses Azure.

```md
## Azure

- Before any `az` command, compare `az account show` with the tenant, subscription and user in `AGENTS.local.md`. On a mismatch stop and tell the user; logging in is the user's job.
- Azure DevOps: org `<org url>`, project `<project>`. Pass both explicitly on every `az devops`, `az boards` and `az repos` call.
- Azure resources (Cosmos DB, databases, storage…) are read-only: any write needs the user's explicit confirmation, whatever the git permissions say.
```

`AGENTS.local.md`:

```md
- Azure: tenant `<tenant>`, subscription `<subscription>`, user `<user>`
```

### Language

- Default: everything in English.

```md
## Language

<"Everything in English: chat, code, comments, commits." or the user's split>
```
