# agents-repo-setup

Stop re-explaining your workflow to coding agents at the start of every session.

`agents-setup` is a skill you run once inside a repo. It asks a few questions, each with a default you can keep or change, and it writes a compact block of rules (about 40 lines) that every agent reads from then on: which skills to use when, which GitHub account, what the agent may do with git, the Azure context, subagent models, language.

Everything lives in the repo: no global instructions, because accounts, Azure tenants and skills differ from repo to repo.

## How the setup runs

1. **Explore**: reads the repo and your machine without changing anything: installed agents, existing `AGENTS.md` / `CLAUDE.md`, git remote, credential helper and identity, `gh` accounts, `az account show`, installed skills and plugins.
2. **Ask**: one chat message with eight numbered questions; each explains what it decides, lists the options with a default and an open last option, and "ok" keeps every default:
   1. which agents work in the repo (Claude Code, Codex, …): the rest adapts to them
   2. local or shared (rules kept in your uncommitted `AGENTS.local.md`, or committed in `AGENTS.md`)
   3. skills and plugins: keep the default ones, drop some, add others you have installed (grouped by source)
   4. subagent model and effort, per agent
   5. GitHub account for this repo, when the remote is on GitHub (asked even with one account)
   6. git permissions: read-only / local commits free, push and PRs on confirmation / everything except pushing to main
   7. Azure, in two independent parts: resources (tenant, subscription, user) and Azure DevOps (org and project of the work items)
   8. language for chat and for code
3. **Prerequisites**: anything missing is installed after your ok; a default plugin you dropped is switched off for this repo only.
4. **Write**: shows every file (or the diff on a re-run) and writes after your ok.
5. **Finish**: reminds you to run `setup-matt-pocock-skills` if the repo needs it.

## What it writes

`AGENTS.md`, the rules every agent reads (shortened):

```md
<!-- agents-repo-setup -->
# Agent instructions

Standing rules for every agent working in this repo. Personal values (GitHub account, Azure identity) are in `AGENTS.local.md`.

## Workflow

New features go through the mattpocock/skills pipeline, one step at a time. The user starts each step: when one finishes, suggest the next instead of running it yourself.

1. `grill-with-docs`: interview the user until the design is settled; updates `GLOSSARY.md` and ADRs.
2. `to-spec`: turn the settled design into a spec on the issue tracker.
3. `to-tickets`: split the spec into tickets with blocking relations.
4. `implement-spec`: implement the tickets with subagents, TDD and a final review (`code-review` plus `ponytail-review`).

## Code

Write and review code the ponytail way: the plugin loads it every session.

- Final review of `implement-spec`: run `code-review` and `ponytail-review` on the integration branch in parallel, as separate axes; keep their reports separate, then fix every finding of `code-review` and the "Must fix" and "Should fix" of `ponytail-review` in one implementer subagent. List the "Nice to have" for the user.
- Reviews outside the pipeline: `ponytail-review`.

## Subagents

Subagents use the model and effort below: pass both explicitly at every spawn.

- Claude Code: `model: sonnet`; effort `high` for implementation and review, `medium` for research and exploration.

## Git

- Allowed without asking: creating branches, local commits.
- Ask first: push, opening or merging PRs, merging to main, creating or editing issues.

This machine's active GitHub account, which may not be this repo's, is used by `gh` and by git through its `gh` credential helper. Prefix every `gh` command, and every `git` command that reaches GitHub (push, pull, fetch), with `GH_TOKEN=$(gh auth token -u <account>)`, `<account>` from `AGENTS.local.md`; if it is missing, ask the user.

## Language

Everything in English: chat, code, comments, commits.

## Personal instructions

Before your first reply or action, read `AGENTS.local.md` next to this file, if it exists, and follow it as part of these instructions: @AGENTS.local.md
It holds this checkout's personal rules; it is not committed and not every agent loads it on its own. …
<!-- /agents-repo-setup -->
```

`AGENTS.local.md`, your personal values, never committed:

```md
<!-- agents-repo-setup -->
- GitHub account: `your-account`
- Azure user: `…`
<!-- /agents-repo-setup -->
```

Two files, never a `CLAUDE.md`: `AGENTS.md` holds what belongs to the project, `AGENTS.local.md` what belongs to you. `AGENTS.md` always ends with the pointer above: Claude Code loads `AGENTS.local.md` through the `@` import, other agents by following the sentence.

| Rules | `AGENTS.md` | `AGENTS.local.md` (never committed) |
|---|---|---|
| shared | rules + pointer, committed | personal values |
| local | pointer only | rules + personal values |

In a local repo a new `AGENTS.md` is committed only if you say so, so tools like `setup-matt-pocock-skills`, which write there, can be shared while your rules stay yours. If the repo already has a `CLAUDE.md`, Claude Code reads only that file: the setup offers to add `@AGENTS.md` to it, or to switch Claude Code to read both.

In a **workspace**, a folder that holds several repos and where you launch the agents, nothing is committed: the rules go to the folder's `AGENTS.md`, your values to `AGENTS.local.md`, and git commands run inside the sub-repos.

If an instruction file already has text of its own, the setup lists what repeats or contradicts the new block and asks how to merge it. Re-running the setup replaces only the text between the markers.

## Default configuration

- [mattpocock/skills](https://github.com/mattpocock/skills): `/grill-with-docs` → `/to-spec` → `/to-tickets` → `/implement-spec`, with `/setup-matt-pocock-skills` at the start of a project
- [ponytail](https://github.com/DietrichGebert/ponytail) for code
- the subagent, git, Azure and language rules above

To change the defaults, fork this repo and edit [`skills/agents-setup/defaults.md`](skills/agents-setup/defaults.md): each entry holds when to use it, how to install it, and the exact line written to `AGENTS.md`.

## Install

```sh
npx skills add gianmadd/agents-repo-setup -g
```

Installs for every agent the [`skills`](https://github.com/vercel-labs/skills) CLI supports (Claude Code, Codex, Cursor, Copilot…).

## Use

Open an agent in the repo and start the `agents-setup` skill (`/agents-setup` in Claude Code, `$agents-setup` in Codex).

## Files

- [`skills/agents-setup/SKILL.md`](skills/agents-setup/SKILL.md): the setup steps
- [`skills/agents-setup/defaults.md`](skills/agents-setup/defaults.md): the default configuration, the pointer and the exact text each section writes
- [`skills/agents-setup/agents/openai.yaml`](skills/agents-setup/agents/openai.yaml): keeps Codex from starting the setup on its own
- [`GLOSSARY.md`](GLOSSARY.md): project vocabulary
