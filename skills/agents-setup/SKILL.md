---
name: agents-setup
description: "Set up this repo's agent instructions (AGENTS.md) from a recommended configuration the user confirms section by section."
disable-model-invocation: true
---

# Agents setup

Write this repo's **repo instructions**: a short `AGENTS.md` block telling any coding agent which skills to use when, plus the git, Azure, subagent and language rules for this repo. The recommendation lives in [recommended.md](recommended.md). You propose, the user decides: every section is confirmed before anything is written.

The written block stays small (about 20 lines): it is loaded on every turn of every session. Rules only, no repo description, no commands the agent can discover itself.

## 1. Explore

Read-only. Gather:

- `AGENTS.md`, `CLAUDE.md`, `CLAUDE.local.md` at the repo root, and whether any holds an `<!-- agents-repo-setup -->` block. If one does, this is a re-run: its current choices become the proposals in step 2.
- `git remote -v`, `git config user.email`, `gh auth status` (the logged-in accounts).
- `az account show`, if `az` is installed.
- Installed skills and plugins: `npx skills ls -g`, `~/.claude/skills/`, `.claude/skills/` and `.agents/skills/` in the repo, `claude plugin list`.
- Whether `docs/agents/` exists (`/setup-matt-pocock-skills` has run).

Done when every item above has an answer, "absent" included.

## 2. Propose and confirm

Present every section below in one message, numbered, each with your recommendation, worded so "ok" accepts it. The user answers per number; re-ask what they changed until every section is confirmed.

1. **Local or shared repo?** Shared: the instructions are committed. Local: they stay out of git.
2. **Skills and plugins**: the entries of recommended.md, each marked present or missing. Ask whether to keep them all, drop some, or add others; to add, list installed skills and plugins outside the recommendation with their descriptions, and ask when each should be used.
3. **Subagents**: the recommended models and effort.
4. **GitHub account**: list the accounts from `gh auth status` and ask which one this repo uses, even when there is only one. Show `git config user.email` beside it; when it does not match the chosen account (a GitHub noreply email carries the account name), warn the user. Never change git config.
5. **Git permissions**: the three levels, recommending (b).
6. **Azure**: does this repo use Azure? If yes, propose tenant, subscription and user from `az account show`, and ask for the Azure DevOps org and project.
7. **Language**: chat language, and language of code, comments and commits. Recommend English for both.

Done when the user has confirmed all seven.

## 3. Prerequisites

For each kept entry, check its prerequisites against what step 1 found. For each missing one, show the install command from recommended.md and run it only after the user's explicit ok. Re-check after installing.

Done when every kept entry is installed, or the user chose to go ahead without it.

## 4. Write

Build the block from the confirmed sections, in this shape, using the lines from recommended.md:

```md
<!-- agents-repo-setup -->
# Agent instructions

## Workflow
<one line per kept skill or plugin>

<rule sections: Subagents, Git, Azure, Language>
<!-- /agents-repo-setup -->
```

Choose the destination:

| Repo | Existing files | Write to |
|---|---|---|
| shared | none | new `AGENTS.md` |
| shared | `AGENTS.md` | append the block to it |
| local | no `AGENTS.md` | new `AGENTS.md`, then add `AGENTS.md` to `.git/info/exclude` |
| local | team `AGENTS.md` | `CLAUDE.local.md`, starting with the line `@AGENTS.md` so Claude still reads the team file |

When a `CLAUDE.md` exists and nothing imports `AGENTS.md` yet, Claude Code will skip `AGENTS.md` (it reads it on its own only when no `CLAUDE.md` exists). Shared repo: add `@AGENTS.md` as the first line of `CLAUDE.md`. Local repo: put that line in `CLAUDE.local.md` and add it to `.git/info/exclude` too; a local setup leaves committed files untouched.

On a re-run, replace only the text between the two markers. Show the user the full block, or the diff on a re-run, and write after their ok.

## 5. Finish

If the feature workflow was kept and `docs/agents/` is missing, tell the user to run `/setup-matt-pocock-skills` now: it adds its own section to the same `AGENTS.md`.

Report what was written, where, and anything the user skipped.
