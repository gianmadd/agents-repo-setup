---
name: agents-setup
description: "Set up this repo's agent instructions (AGENTS.md) from a default configuration the user adjusts question by question."
disable-model-invocation: true
---

# Agents setup

Write this repo's **repo instructions**: a rules block telling any coding agent which skills to use when, plus the git, Azure, subagent and language rules for this repo, and the user's personal values. Two files only, never a `CLAUDE.md`: `AGENTS.md` for what belongs to the project, `AGENTS.local.md`, never committed, for what belongs to the user. The defaults live in [defaults.md](defaults.md). You ask, the user decides: nothing is written before every question is answered.

The block is loaded on every turn of every session: rules and the reason behind each, nothing the agent can discover by looking at the repo.

## 1. Explore

Read-only. Gather:

- Agents installed: `which claude codex cursor gemini copilot`.
- `AGENTS.md`, `AGENTS.local.md`, `CLAUDE.md`, `CLAUDE.local.md` at the repo root, whether `AGENTS.md` is tracked (`git ls-files AGENTS.md`), and whether any holds an `<!-- agents-repo-setup -->` block. If one does, this is a re-run: its current choices become the defaults in step 2. A block in `CLAUDE.local.md` comes from an older version of this setup: step 4 moves it.
- `git remote -v` (host: GitHub or not; protocol: HTTPS or SSH), `git config --get-all credential.helper` and `git config --get-all credential.https://github.com.helper`, `git config user.email`, `gh auth status` (the logged-in accounts).
- `az account show`, if `az` is installed.
- Installed skills and plugins: `npx skills ls -g`, `.agents/skills/` and `.claude/skills/` in the repo, `claude plugin list` and `~/.codex/config.toml` for the agents present.
- Whether `docs/agents/` exists (`setup-matt-pocock-skills` has run).

Done when every item above has an answer, "absent" included.

## 2. Ask

Ask every question in one chat message, numbered, in plain text (no multiple-choice tool). For each question:

- explain in one or two sentences what it decides and what changes with each answer, for a user who has never seen this setup;
- list the options lettered, mark the default "(default)", and end with an open option ("other: tell me");
- keep your own opinion out of it.

The user answers per number, and "ok" takes every default. Follow up on anything unclear until every answer is settled. On a re-run, the current choice is the default. The agents chosen in question 1 shape the others: install commands, switch-offs and subagent lines cover only those agents.

1. **Agents**: Which agents work in this repo? Options: the installed ones. Default: all installed.
2. **Local or shared**: Should the rules be committed for the team (shared: in `AGENTS.md`) or stay yours (local: in `AGENTS.local.md`)? Personal values stay in `AGENTS.local.md` either way. Default: local.
   Local, and no `AGENTS.md` exists yet: should the new `AGENTS.md` be committed? It holds the pointer to `AGENTS.local.md` now, and whatever other tools add later (`setup-matt-pocock-skills` writes its section there). Default: no.
3. **Skills and plugins**: Which of the default skills and plugins should this repo use? List each entry of defaults.md, marked installed or missing per agent. Default: all.
   Then: Do you want to add others? List the installed skills and plugins outside the defaults grouped by source: one group per skill package in `npx skills ls -g` (e.g. mattpocock/skills), one per plugin, one for the repo's own skills; each name with a few-word description. For each one added, ask when it is used and whether the user starts it.
4. **Subagents**: Which model and effort should subagents use, per agent? Default: from defaults.md.
5. **GitHub account**: Only when the remote is on GitHub; otherwise skip it and leave out the GitHub lines. Which GitHub account does this repo use? Options: the accounts in `gh auth status`; ask even when there is only one. Default: the one matching `git config user.email` (a GitHub noreply email carries the account name), else the active one. When the chosen account and `git config user.email` disagree, warn the user; git config is theirs to change. When the remote is SSH or git's credential helper is not `gh auth git-credential`, tell the user the rule covers only `gh`: git authenticates with their SSH key or helper.
6. **Git permissions**: What may the agent do with git and GitHub without asking? Options: the three levels in defaults.md. Default: (b).
7. **Azure**: Does this repo use Azure? Default: no. If yes: tenant, subscription and user (default: from `az account show`), Azure DevOps org and project.
8. **Language**: Which language for chat, and which for code, comments and commits? Default: English for both.

Done when all eight are answered (seven without a GitHub remote).

## 3. Prerequisites

For each kept entry and each agent in use, check its prerequisites against what step 1 found. For each missing one, show the install command from defaults.md and run it only after the user's explicit ok. Re-check after installing.

For a default plugin the user dropped but has installed, offer its switch-off for this repo from defaults.md, and write it after the user's ok.

Done when every kept entry is installed or knowingly skipped, and every dropped plugin is switched off or knowingly left on.

## 4. Write

Build the blocks from the answers, each between the markers:

```md
<!-- agents-repo-setup -->
…
<!-- /agents-repo-setup -->
```

The rules block is the header followed by the kept sections, in defaults.md order. Where each part goes:

| Rules | `AGENTS.md` block | `AGENTS.local.md` block |
|---|---|---|
| shared | rules block, then the pointer | personal values |
| local | the pointer | rules block, then personal values |

`AGENTS.md`: append the block to an existing file, else create it. A new one is committed in a shared repo, and in a local repo only if the user said so in question 2; otherwise add it to `.git/info/exclude`. In a local repo with a team `AGENTS.md`, the pointer is a committed change to the team's file: tell the user before writing.

`AGENTS.local.md` goes at the repo root and into `.git/info/exclude`. A switch-off file you create in a local repo, or in a shared repo where it is not already tracked, goes into `.git/info/exclude` as well.

When a `CLAUDE.md` exists, Claude Code reads it instead of `AGENTS.md` unless it imports `AGENTS.md`. Never create a `CLAUDE.md`; if one exists without the import, offer: (a) `@AGENTS.md` as its first line, a committed change (default in a shared repo); (b) the user sets **Project instructions** to `claude-md-and-agents-md` in Claude Code's `/config`, which applies to all their repos (default in a local repo).

On a re-run, replace only the text between the markers. A block left in `CLAUDE.local.md` by an older version: remove it, delete the file if nothing else is left, and drop its line from `.git/info/exclude`. Show the user every file's full block, or the diff on a re-run, and write after their ok.

## 5. Finish

If the feature workflow was kept and `docs/agents/` is missing, tell the user to run `setup-matt-pocock-skills` now: it writes its section to `AGENTS.md` (to `CLAUDE.md` if one exists), so that section is tracked exactly when that file is. Its new files (`docs/agents/`, `GLOSSARY.md`, `docs/adr/`) start untracked: the user commits them or adds them to `.git/info/exclude`.

If the rules are local and Codex is in use, tell the user Codex loads them only by following the pointer sentence, not on its own.

If a Codex switch-off was written and the project is not trusted in `~/.codex/config.toml`, tell the user Codex ignores `.codex/config.toml` until they trust the project.

Report what was written, where, and anything the user skipped.
