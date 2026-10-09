# agents-repo-setup

Stop re-explaining your workflow to coding agents at the start of every session.

`agents-setup` is a skill you run once inside a repo. It proposes a recommended configuration, you confirm or change each section, and it writes a short `AGENTS.md` block (about 20 lines) that every agent reads from then on: which skills to use when, which GitHub account, what the agent may do with git, the Azure context, subagent models, language.

Everything lives in the repo: no global instructions, because accounts, Azure tenants and skills differ from repo to repo.

## How the setup runs

1. **Explore**: reads the repo and your machine without changing anything: existing `AGENTS.md` / `CLAUDE.md`, git remote and identity, `gh` accounts, `az account show`, installed skills and plugins.
2. **Propose**: one message with seven numbered sections, each with a recommendation you accept with "ok" or change:
   1. local or shared repo (instructions kept out of git, or committed)
   2. skills and plugins: keep the recommended ones, drop some, add others you have installed
   3. subagent models and effort
   4. GitHub account for this repo (always asked, even with one account)
   5. git permissions: read-only / local commits free, push and PR on confirmation / everything except pushing to main
   6. Azure: tenant, subscription, user, Azure DevOps org and project, if the repo uses Azure
   7. language for chat and for code
3. **Prerequisites**: anything missing is shown with its install command and installed after your ok.
4. **Write**: shows the block (or the diff on a re-run) and writes it after your ok.
5. **Finish**: reminds you to run `/setup-matt-pocock-skills` if the repo needs it.

## What it writes

```md
<!-- agents-repo-setup -->
# Agent instructions

## Workflow
- New feature: suggest the user runs, one at a time, `/grill-with-docs` → `/to-spec` → `/to-tickets` → `/implement-spec`. …
- Non-trivial task asked outside a skill workflow: ask once whether to use the matching skill instead. …
- Code: follow ponytail. Agents without it: the smallest complete change, no unrequested abstractions.

## Subagents (Claude Code)
- `model: sonnet`; effort `high` for implementation and review, `medium` for research and exploration.

## Git
- Run gh as `GH_TOKEN=$(gh auth token -u <account>) gh …`.
- Branches and local commits allowed; push, PR, merge to main and ticket changes need explicit confirmation.

## Language
- Everything in English.
<!-- /agents-repo-setup -->
```

Where it goes:

| Repo | Written to |
|---|---|
| shared | `AGENTS.md`, committed |
| local | `AGENTS.md`, excluded through `.git/info/exclude` |
| local, team already has an `AGENTS.md` | `CLAUDE.local.md`, importing the team's `AGENTS.md` |

Re-running the setup replaces only the text between the markers.

## Recommended configuration

- [mattpocock/skills](https://github.com/mattpocock/skills): `/grill-with-docs` → `/to-spec` → `/to-tickets` → `/implement-spec`, with `/setup-matt-pocock-skills` at the start of a project
- [ponytail](https://github.com/DietrichGebert/ponytail) for code
- the git, Azure, subagent and language rules above

To recommend something else, fork this repo and edit [`skills/agents-setup/recommended.md`](skills/agents-setup/recommended.md): each entry holds when to use it, how to install it, and the exact line written to `AGENTS.md`.

## Install

```sh
npx skills add gianmadd/agents-repo-setup -g
```

Installs for every agent the [`skills`](https://github.com/vercel-labs/skills) CLI supports (Claude Code, Codex, Cursor, Copilot…).

## Use

Open an agent in the repo and run `/agents-setup`.

## Files

- [`skills/agents-setup/SKILL.md`](skills/agents-setup/SKILL.md): the setup steps
- [`skills/agents-setup/recommended.md`](skills/agents-setup/recommended.md): the recommended configuration
- [`GLOSSARY.md`](GLOSSARY.md): project vocabulary
