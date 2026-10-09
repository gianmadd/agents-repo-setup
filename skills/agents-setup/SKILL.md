---
name: agents-setup
description: "Set up this repo's agent instructions (AGENTS.md) from a default configuration the user adjusts question by question."
disable-model-invocation: true
---

# Agents setup

Write this repo's **repo instructions**: an `AGENTS.md` block telling any coding agent which skills to use when, plus the git, Azure, subagent and language rules for this repo, and an uncommitted `AGENTS.local.md` with the user's personal values. The defaults live in [defaults.md](defaults.md). You ask, the user decides: nothing is written before every question is answered.

The block is loaded on every turn of every session: rules and the reason behind each, nothing the agent can discover by looking at the repo.

## 1. Explore

Read-only. Gather:

- Agents installed: `which claude codex cursor gemini copilot`.
- `AGENTS.md`, `AGENTS.local.md`, `CLAUDE.md`, `CLAUDE.local.md` at the repo root, and whether any holds an `<!-- agents-repo-setup -->` block. If one does, this is a re-run: its current choices become the defaults in step 2.
- `git remote -v`, `git config user.email`, `gh auth status` (the logged-in accounts).
- `az account show`, if `az` is installed.
- Installed skills and plugins: `npx skills ls -g`, `.agents/skills/` and `.claude/skills/` in the repo, `claude plugin list` and `~/.codex/config.toml` for the agents present.
- Whether `docs/agents/` exists (`setup-matt-pocock-skills` has run).

Done when every item above has an answer, "absent" included.

## 2. Ask

Each question is a plain question with its options and the default marked "(default)"; keep your own opinion out of it.

- With a multiple-choice question tool (Claude Code: `AskUserQuestion`), use it: default option first, a free-text answer as the open last option (the tool adds "Other" itself), multi-select where several answers fit (agents, skills). It takes at most 4 questions per call and 4 options per question: ask in batches, and when a list is longer (skills to add), ask first by group, then within the chosen groups.
- Without one, ask every question in one message, numbered, each option lettered and an open last option; "ok" takes every default.

Follow up on anything unclear until every answer is settled. On a re-run, the current choice is the default. The agents chosen in question 1 shape the others: install commands, switch-offs and subagent lines cover only those agents.

1. **Agents**: Which agents work in this repo? Options: the installed ones. Default: all installed.
2. **Local or shared**: Should the instructions stay out of git (local) or be committed for the team (shared)? Default: local.
3. **Skills and plugins**: Which of the default skills and plugins should this repo use? List each entry of defaults.md, marked installed or missing per agent. Default: all.
   Then: Do you want to add others? List the installed skills and plugins outside the defaults grouped by source: one group per skill package in `npx skills ls -g` (e.g. mattpocock/skills), one per plugin, one for the repo's own skills; each name with a few-word description. For each one added, ask when it is used and whether the user starts it.
4. **Subagents**: Which model and effort should subagents use, per agent? Default: from defaults.md.
5. **GitHub account**: Which GitHub account does this repo use? Options: the accounts in `gh auth status`; ask even when there is only one. Default: the one matching `git config user.email` (a GitHub noreply email carries the account name), else the active one. When the chosen account and `git config user.email` disagree, warn the user; git config is theirs to change.
6. **Git permissions**: What may the agent do with git and GitHub without asking? Options: the three levels in defaults.md. Default: (b).
7. **Azure**: Does this repo use Azure? Default: no. If yes: tenant, subscription and user (default: from `az account show`), Azure DevOps org and project.
8. **Language**: Which language for chat, and which for code, comments and commits? Default: English for both.

Done when all eight are answered.

## 3. Prerequisites

For each kept entry and each agent in use, check its prerequisites against what step 1 found. For each missing one, show the install command from defaults.md and run it only after the user's explicit ok. Re-check after installing.

For a default plugin the user dropped but has installed, offer its switch-off for this repo from defaults.md, and write it after the user's ok.

Done when every kept entry is installed or knowingly skipped, and every dropped plugin is switched off or knowingly left on.

## 4. Write

Build the block from the answers, in defaults.md order, between the markers:

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

Write the personal values (`AGENTS.local.md` blocks in defaults.md) to `AGENTS.local.md` at the repo root, inside the same markers, and add it to `.git/info/exclude`. A switch-off file you create in a local repo, or in a shared repo where it is not already tracked, goes into `.git/info/exclude` too.

On a re-run, replace only the text between the markers. Show the user every file's full block, or the diff on a re-run, and write after their ok.

## 5. Finish

If the feature workflow was kept and `docs/agents/` is missing, tell the user to run `setup-matt-pocock-skills` now: it adds its own section to the same instructions file.

If a Codex switch-off was written and the project is not trusted in `~/.codex/config.toml`, tell the user Codex ignores `.codex/config.toml` until they trust the project.

Report what was written, where, and anything the user skipped.
