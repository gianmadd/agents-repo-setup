# agents-repo-setup

Stop re-explaining your workflow to coding agents at the start of every session.

`agents-setup` is a skill you run once per repo. It asks up to ten questions, each with a default, and writes about 40 lines of rules that Claude Code, Codex and the other agents read in every session: which skills to use and when, which GitHub account, what the agent may do with git and how it names branches and writes commits, the Azure context, subagent models, language. Everything is per repo, because accounts, Azure tenants and skills differ from repo to repo.

## Quick start

```sh
npx skills add gianmadd/agents-repo-setup -g
```

Then open an agent in the repo and run `/agents-setup` (Claude Code) or `$agents-setup` (Codex). Answer "ok" to keep every default.

To update the skill later:

```sh
npx skills update agents-setup -g
```

The Workflow's skills live in each repo; update them there:

```sh
npx skills update -p
```

## What you get

A Workflow for new features, built on [mattpocock/skills](https://github.com/mattpocock/skills) and on [gianmadd/agents-flow](https://github.com/gianmadd/agents-flow), three planning steps copied from [salvatorebottiglieri/agentic-dev-kit](https://github.com/salvatorebottiglieri/agentic-dev-kit) that take their git, tracker and UI rules from your `AGENTS.md` (why a copy: [ADR 0002](docs/adr/0002-workflow-kit-is-agents-flow.md)). [ponytail](https://github.com/DietrichGebert/ponytail) keeps the code lean. The Workflow's skills install in the repo, not globally:

| You want to… | You do | What happens |
|---|---|---|
| build a feature | start `/grill-with-docs` | the agent interviews you until the design is settled, then suggests each next step; you start it |
| | `/to-spec` | the design becomes a spec on the issue tracker |
| | `/product-review` | an HTML mockup of the UI, per persona and edge case (only with the web UI check) |
| | `/system-architecture` | diagrams, contracts and domain invariants on the spec; skipped for small changes |
| | `/to-tickets` | the spec becomes tickets, with what blocks what |
| | `/program-design` | the signatures and files each ticket will touch |
| | `/implement-spec` | subagents build the tickets in parallel with TDD, each in its own worktree; then a final review on two axes (`code-review`, `ponytail-review`); push and PR as your git rules say |
| fix something small, ask a question | just ask | for a larger task the agent asks once whether to use the Workflow |
| keep the code lean | nothing | ponytail is always on: the least code that fully solves the task |
| check a UI change | nothing | before saying it's done, the agent runs the app, in the browser with [playwright-cli](https://github.com/microsoft/playwright-cli) on the Android emulator or iOS simulator with [Maestro](https://maestro.dev), or on the Mac with [Peekaboo](https://github.com/openclaw/Peekaboo), tries the flow and reads the errors |

First time in a repo, run `/setup-matt-pocock-skills` too: it tells the Workflow where the issues live. The setup reminds you.

## The questions

One message, every question with its options and a default:

| # | Decides |
|---|---|
| 1 | which agents work in the repo |
| 2 | if the folder isn't a git repo yet, whether to `git init`; then rules shared with the team (committed) or yours only |
| 3 | which skills and plugins, and any others you have installed |
| 4 | how the agent checks UI changes running: web with `playwright-cli`, Android or iOS with Maestro, native macOS apps with Peekaboo, any combination or none; only what you pick gets installed; always asked |
| 5 | coordinator model; subagent model and effort |
| 6 | install the `fns` launcher, unless it is already installed |
| 7 | GitHub account, or "not on GitHub"; with one account, just a confirmation |
| 8 | what the agent may do with git without asking; how it names branches and writes commits and PRs |
| 9 | Azure resources and Azure DevOps |
| 10 | language for chat, and for code and commits |

Before writing, it lists what goes in each file and waits for your ok. Anything missing is installed only after your ok.

## Workers across subscriptions

The setup offers to install `fns` from [gianmadd/agents-runner](https://github.com/gianmadd/agents-runner): before each session, it picks which account runs it and which account runs each worker the skills spawn.

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
- **Background permissions**: with the Workflow, `.claude/settings.local.json` pre-approves the commands `implement-spec`'s background agents need, since they can't ask you: file edits, basic file reading (`ls`, `cat`, `grep`…), stopping the servers they start (`lsof`, `kill`), the repo's test, build and run commands, `playwright-cli`, `maestro`, `adb`, `xcrun simctl`, `peekaboo` or `xcodebuild` for the UI check, as chosen, and only the git commands they use (`status`, `diff`, `log`, `show`, `rev-parse`, `branch`, `switch`, `checkout -b`, `add`, `commit`, `merge`, `worktree`). Every other git command asks, push, `reset --hard`, `clean`, `branch -D`, `rebase`, `commit --amend` and the forms of `switch`, `checkout`, `restore` and `stash` that discard changes included. With git level (a) only reading git is pre-approved; with (c), push too, except force-push and push to `main`; with (d), every git command.
- **Re-run**: the setup changes only the text between its `<!-- agents-repo-setup -->` markers.

<details>
<summary>Example <code>AGENTS.md</code> (shared rules, shortened)</summary>

```md
<!-- agents-repo-setup -->
# Agent instructions

Standing rules for every agent working here. Personal values are in `AGENTS.local.md`.

## Workflow

New features go through the Workflow, one step at a time. The user starts each step: when one finishes, suggest the next instead of running it yourself, and say which optional step you would skip and why. …

1. `grill-with-docs`: interview the user until the design is settled; updates `GLOSSARY.md` and ADRs. Before the spec, have a subagent try to disprove what the conversation assumed about the current code: …
2. `to-spec`: turn the settled design into a spec on the issue tracker.
3. `product-review`, when the feature touches UI: …
4. `system-architecture`, unless it is a bug fix or a trivial feature: …
5. `to-tickets`: split the spec into tickets with blocking relations.
6. `program-design`, unless no signature changes: …
7. `implement-spec`: implement the tickets with subagents and TDD, then a final code review.

## Code

Write and review code the ponytail way: the plugin loads it every session; where ponytail is installed as a skill instead, load the `ponytail` skill before writing code.

- Final review of `implement-spec`: run `code-review` and `ponytail-review` on the integration branch in parallel, as separate axes; …
- Reviews outside the Workflow: `ponytail-review`.

## UI check

Before calling a UI change done, check it running: passing tests don't show a blank screen or a hidden button. When done, stop the processes you started: a server left running holds its port next time.

- Web: start the app (find how from the repo), open it with `playwright-cli`, walk through the changed flow and check the console for errors. …

## Subagents

Subagents use the model and effort below: pass both explicitly at every spawn.

- Claude Code: `model: sonnet`; effort `medium`.

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
