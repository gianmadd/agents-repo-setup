# agents-repo-setup

Stop re-explaining your workflow to coding agents at the start of every session.

`agents-setup` is a skill you run once per repo. It asks eight questions, each with a default, and writes about 40 lines of rules that Claude Code, Codex and the other agents read in every session: which skills to use and when, which GitHub account, what the agent may do with git and how it names branches and writes commits, the Azure context, subagent models, language. Everything is per repo, because accounts, Azure tenants and skills differ from repo to repo.

## Quick start

```sh
npx skills add gianmadd/agents-repo-setup -g
```

Then open an agent in the repo and run `/agents-setup` (Claude Code) or `$agents-setup` (Codex). Answer "ok" to keep every default.

To update the skill later:

```sh
npx skills update agents-setup -g
```

## What you get

A feature workflow built on [mattpocock/skills](https://github.com/mattpocock/skills), with [ponytail](https://github.com/DietrichGebert/ponytail) for the code:

| You want to… | You do | What happens |
|---|---|---|
| build a feature | start `/grill-with-docs` | the agent interviews you until the design is settled, then suggests each next step; you start it |
| | `/to-spec` | the design becomes a spec on the issue tracker |
| | `/to-tickets` | the spec becomes tickets, with what blocks what |
| | `/implement-spec` | subagents build the tickets with TDD; the final review runs `code-review` and `ponytail-review` on its own |
| fix something small, ask a question | just ask | for a larger task the agent asks once whether to use the workflow |
| keep the code lean | nothing | ponytail is always on: the least code that fully solves the task |
| check a UI change | nothing | before saying it's done, the agent opens the app with [playwright-cli](https://github.com/microsoft/playwright-cli), tries the flow and reads the console |

First time in a repo, run `/setup-matt-pocock-skills` too: it tells the workflow where the issues live. The setup reminds you.

## The questions

One message, every question with its options and a default:

| # | Decides | Default |
|---|---|---|
| 1 | which agents work in the repo | all installed |
| 2 | if the folder isn't a git repo yet, whether to `git init`; then rules shared with the team (committed) or yours only | yours only |
| 3 | which skills and plugins, and any others you have installed | the workflow, ponytail and the UI check (off in repos with code but no web UI) |
| 4 | subagent model and effort | Claude Code `sonnet`, Codex `gpt-6-luna` |
| 5 | GitHub account, or "not on GitHub"; with one account, just a confirmation | the one matching your git email |
| 6 | what the agent may do with git without asking; how it names branches and writes commits and PRs | branches and local commits; `feat/420-login-page`, short explanatory messages |
| 7 | Azure resources and Azure DevOps | no |
| 8 | language for chat, and for code and commits | English |

Before writing, it lists what goes in each file and waits for your ok. Anything missing is installed only after your ok.

## Where the rules go

Two files:

- `AGENTS.md`: what belongs to the project.
- `AGENTS.local.md`: what belongs to you (GitHub account, Azure user, chat language). Never committed.

| You chose | `AGENTS.md` | `AGENTS.local.md` |
|---|---|---|
| shared | the rules, committed | your values |
| yours only | a pointer to `AGENTS.local.md` | the rules and your values |

`AGENTS.md` always ends with a pointer that makes every agent read `AGENTS.local.md` too.

- **Existing `CLAUDE.md`**: Claude Code reads only that file, so the setup offers to add `@AGENTS.md` to it.
- **Existing instructions**: what they already say becomes the default answer (e.g. "commits in Italian, from your `CLAUDE.md`"), so the new rules don't contradict them; what they repeat, the setup offers to delete, so it isn't loaded twice.
- **Workspace** (a folder holding several repos): both files go in the folder, nothing is committed, and each sub-repo can have its own GitHub account.
- **Re-run**: the setup changes only the text between its `<!-- agents-repo-setup -->` markers.

<details>
<summary>Example <code>AGENTS.md</code> (shared rules, shortened)</summary>

```md
<!-- agents-repo-setup -->
# Agent instructions

Standing rules for every agent working here. Personal values are in `AGENTS.local.md`.

## Workflow

New features go through the mattpocock/skills pipeline, one step at a time. The user starts each step: when one finishes, suggest the next instead of running it yourself.

1. `grill-with-docs`: interview the user until the design is settled; updates `GLOSSARY.md` and ADRs.
2. `to-spec`: turn the settled design into a spec on the issue tracker.
3. `to-tickets`: split the spec into tickets with blocking relations.
4. `implement-spec`: implement the tickets with subagents, TDD and a final code review.

## Code

Write and review code the ponytail way: the plugin loads it every session.

- Final review of `implement-spec`: run `code-review` and `ponytail-review` on the integration branch in parallel, as separate axes; …
- Reviews outside the pipeline: `ponytail-review`.

## UI check

Before calling a UI change done, start the app (find how from the repo), open it with `playwright-cli`, walk through the changed flow and check the console for errors. …

## Subagents

Subagents use the model and effort below: pass both explicitly at every spawn.

- Claude Code: `model: sonnet`; effort `high` for implementation and review, `medium` for research and exploration.

## Git

- Allowed without asking: creating branches, local commits.
- Ask first: push, opening or merging PRs, merging to main, creating or editing work items.
- Always ask first, whatever the above allows: `push --force`, `reset --hard`, `branch -D`, `clean`, and rewriting pushed commits. They lose work.
- Branches: `<type>/<task>-<short-title>`, e.g. `feat/420-login-page`; …
- Commit messages and PR descriptions: say what changed and why, in plain words. Short and explanatory, not a wall of technical detail.

Prefix every `gh` command, and every `git` command that reaches GitHub, with `GH_TOKEN=$(gh auth token -u <account>)`, `<account>` from `AGENTS.local.md`; …

## Secrets

Never print, commit or paste secrets: `.env` files, keys, tokens, passwords, connection strings. …

## Azure resources

- They live in tenant `your-tenant`, subscription `your-subscription`. Before any `az` command that touches them (anything but `az devops`, `az boards`, `az repos`, `az pipelines`), check `az account show` against these and the user in `AGENTS.local.md`; …
- Read-only: any write needs the user's explicit confirmation, whatever the git permissions say.

## Azure DevOps

- Work items live in org `https://dev.azure.com/your-org`, project `your-project`; a bare number like "task 420" is a work item there. …
- Reading is free; creating or editing work items needs the user's explicit confirmation.

## Language

Code, comments and commits in English.

## Personal instructions

Before your first reply or action, read `AGENTS.local.md` next to this file, if it exists, and follow it as part of these instructions: @AGENTS.local.md
…
<!-- /agents-repo-setup -->
```

`AGENTS.local.md`:

```md
<!-- agents-repo-setup -->
- GitHub account: `your-account`
- Azure user: `you@example.com`
- Chat language: English
<!-- /agents-repo-setup -->
```

</details>

## Files

- [`skills/agents-setup/SKILL.md`](skills/agents-setup/SKILL.md): the whole setup: the procedure, then one entry per skill or rule with its question, default and the exact text it writes
- [`GLOSSARY.md`](GLOSSARY.md): project vocabulary
