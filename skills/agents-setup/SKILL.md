---
name: agents-setup
description: "Set up this repo's agent instructions (AGENTS.md) from a recommended configuration the user confirms section by section."
disable-model-invocation: true
---

# Agents setup

Write this repo's **repo instructions**: an `AGENTS.md` block telling any coding agent which skills to use when, plus the git, Azure, subagent and language rules for this repo, and an uncommitted `AGENTS.local.md` with the user's personal values. The recommendation lives in [recommended.md](recommended.md). You propose, the user decides: every section is confirmed before anything is written.

The block is loaded on every turn of every session: rules and the reason behind each, nothing the agent can discover by looking at the repo.

## 1. Explore

Read-only. Gather:

- Agents installed: `which claude codex cursor gemini copilot`.
- `AGENTS.md`, `AGENTS.local.md`, `CLAUDE.md`, `CLAUDE.local.md` at the repo root, and whether any holds an `<!-- agents-repo-setup -->` block. If one does, this is a re-run: its current choices become the proposals in step 2.
- `git remote -v`, `git config user.email`, `gh auth status` (the logged-in accounts).
- `az account show`, if `az` is installed.
- Installed skills and plugins: `npx skills ls -g`, `.agents/skills/` and `.claude/skills/` in the repo, `claude plugin list` and `~/.codex/config.toml` for the agents present.
- Whether `docs/agents/` exists (`setup-matt-pocock-skills` has run).

Done when every item above has an answer, "absent" included.

## 2. Propose and confirm

Present every section below in one message, numbered, each with your recommendation, worded so "ok" accepts it. The user answers per number; re-ask what they changed until every section is confirmed. The agents chosen in section 1 shape the others: install commands, switch-offs and subagent lines cover only those agents.

1. **Agents**: which agents work in this repo. Propose the installed ones.
2. **Local or shared repo?** Local: `AGENTS.md` stays out of git. Shared: it is committed. Recommend local.
3. **Skills and plugins**: the entries of recommended.md, each marked installed or missing per agent. Ask whether to keep them all, drop some, or add others; to add, list installed skills and plugins outside the recommendation with their descriptions, and ask when each is used and whether the user starts it.
4. **Subagents**: model and effort for each agent in use.
5. **GitHub account**: list the accounts from `gh auth status` and ask which one this repo uses, even when there is only one. Show `git config user.email` beside it; when it does not match the chosen account (a GitHub noreply email carries the account name), warn the user. Git config is the user's to change.
6. **Git permissions**: the three levels, recommending (b).
7. **Azure**: does this repo use Azure? If yes, propose tenant, subscription and user from `az account show`, and ask for the Azure DevOps org and project.
8. **Language**: chat language, and language of code, comments and commits. Recommend English for both.

Done when the user has confirmed all eight.

## 3. Prerequisites

For each kept entry and each agent in use, check its prerequisites against what step 1 found. For each missing one, show the install command from recommended.md and run it only after the user's explicit ok. Re-check after installing.

For a recommended plugin the user dropped but has installed, propose its switch-off for this repo from recommended.md, and write it after the user's ok.

Done when every kept entry is installed or knowingly skipped, and every dropped plugin is switched off or knowingly left on.

## 4. Write

Build the block from the confirmed sections, in recommended.md order, between the markers:

```md
<!-- agents-repo-setup -->
<header, then the kept sections>
<!-- /agents-repo-setup -->
```

Choose the destination:

| Repo | Existing files | Block goes to |
|---|---|---|
| shared | none | new `AGENTS.md` |
| shared | `AGENTS.md` | appended to it |
| local | no `AGENTS.md` | new `AGENTS.md`, then add `AGENTS.md` to `.git/info/exclude` |
| local | team `AGENTS.md` | `CLAUDE.local.md`, starting with the line `@AGENTS.md` so Claude still reads the team file. Tell the user the other agents won't see the block: Codex has no personal variant that keeps the team file. |

When a `CLAUDE.md` exists and nothing imports `AGENTS.md` yet, Claude Code will skip `AGENTS.md`: it reads it on its own only when no `CLAUDE.md` exists. Shared repo: add `@AGENTS.md` as the first line of `CLAUDE.md`. Local repo: put that line in `CLAUDE.local.md` and add it to `.git/info/exclude` too; a local setup leaves committed files untouched.

Write the personal values (`AGENTS.local.md` blocks in recommended.md) to `AGENTS.local.md` at the repo root, inside the same markers, and add it to `.git/info/exclude`. A switch-off file you create in a local repo, or in a shared repo where it is not already tracked, goes into `.git/info/exclude` too.

On a re-run, replace only the text between the markers. Show the user every file's full block, or the diff on a re-run, and write after their ok.

## 5. Finish

If the feature workflow was kept and `docs/agents/` is missing, tell the user to run `setup-matt-pocock-skills` now: it adds its own section to the same instructions file.

If a Codex switch-off was written and the project is not trusted in `~/.codex/config.toml`, tell the user Codex ignores `.codex/config.toml` until they trust the project.

Report what was written, where, and anything the user skipped.
