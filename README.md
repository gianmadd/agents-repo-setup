# agents-repo-setup

Stop re-explaining your workflow to coding agents at the start of every session.

`agents-setup` is a skill you run once inside a repo. It asks a few questions, each with a default you can keep or change, and it writes a compact `AGENTS.md` block (about 40 lines) that every agent reads from then on: which skills to use when, which GitHub account, what the agent may do with git, the Azure context, subagent models, language.

Everything lives in the repo: no global instructions, because accounts, Azure tenants and skills differ from repo to repo.

## How the setup runs

1. **Explore**: reads the repo and your machine without changing anything: installed agents, existing `AGENTS.md` / `CLAUDE.md`, git remote and identity, `gh` accounts, `az account show`, installed skills and plugins.
2. **Ask**: eight multiple-choice questions, each with a default and an open last option (Claude Code shows them as a picker; other agents as a numbered list where "ok" keeps every default):
   1. which agents work in the repo (Claude Code, Codex, …): the rest adapts to them
   2. local or shared repo (instructions kept out of git, or committed)
   3. skills and plugins: keep the default ones, drop some, add others you have installed (grouped by source)
   4. subagent model and effort, per agent
   5. GitHub account for this repo (always asked, even with one account)
   6. git permissions: read-only / local commits free, push and PRs on confirmation / everything except pushing to main
   7. Azure: tenant, subscription, user, Azure DevOps org and project, if the repo uses Azure
   8. language for chat and for code
3. **Prerequisites**: anything missing is installed after your ok; a default plugin you dropped is switched off for this repo only.
4. **Write**: shows every file (or the diff on a re-run) and writes after your ok.
5. **Finish**: reminds you to run `setup-matt-pocock-skills` if the repo needs it.

## What it writes

`AGENTS.md`, the rules every agent reads (shortened):

```md
<!-- agents-repo-setup -->
# Agent instructions

Standing rules for every agent working in this repo. Personal values (GitHub account, Azure identity) live in `AGENTS.local.md` …

## Workflow

New features go through the mattpocock/skills pipeline, one step at a time. The user starts each step: when one finishes, suggest the next instead of running it yourself.

1. `grill-with-docs`: interview the user until the design is settled; updates `GLOSSARY.md` and ADRs.
2. `to-spec`: turn the settled design into a spec on the issue tracker.
3. `to-tickets`: split the spec into tickets with blocking relations.
4. `implement-spec`: implement the tickets with subagents, TDD and a final code review.

## Code

Write and review code the ponytail way: the plugin loads it every session; use `ponytail-review` for reviews.

## Subagents

Subagents run on a cheaper model than the main session: pass model and effort explicitly at every spawn.

- Claude Code: `model: sonnet`; effort `high` for implementation and review, `medium` for research and exploration.

## Git

`gh` uses whichever account is active on this machine, which may not be this repo's.

- Run every gh command as `GH_TOKEN=$(gh auth token -u <account>) gh …`, with `<account>` from `AGENTS.local.md`.
- Allowed without asking: creating branches, local commits.
- Ask first: push, opening or merging PRs, merging to main, creating or editing issues.

## Language

Everything in English: chat, code, comments, commits.
<!-- /agents-repo-setup -->
```

`AGENTS.local.md`, your personal values, never committed:

```md
<!-- agents-repo-setup -->
- GitHub account: `your-account`
- Azure: tenant `…`, subscription `…`, user `…`
<!-- /agents-repo-setup -->
```

Where the rules go:

| Repo | Written to |
|---|---|
| shared | `AGENTS.md`, committed |
| local | `AGENTS.md`, excluded through `.git/info/exclude` |
| local, team already has an `AGENTS.md` | `CLAUDE.local.md`, importing the team's `AGENTS.md` (Claude Code only) |

Re-running the setup replaces only the text between the markers.

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
- [`skills/agents-setup/defaults.md`](skills/agents-setup/defaults.md): the default configuration and the exact text each section writes
- [`skills/agents-setup/agents/openai.yaml`](skills/agents-setup/agents/openai.yaml): keeps Codex from starting the setup on its own
- [`GLOSSARY.md`](GLOSSARY.md): project vocabulary
