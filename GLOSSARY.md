# Agents Repo Setup

A tool that, run inside a repo, gives it a fixed base of instructions for coding agents, so they don't have to be repeated every session.

## Language

**Setup**:
The skill you run once in a repo; it writes the Repo instructions after checking the Prerequisites.
_Avoid_: installer, bootstrap, init

**Repo instructions**:
The block of rules the Setup writes and every agent reads: in `AGENTS.md` if the repo is shared, in `AGENTS.local.md` if it is local. `AGENTS.md` always holds a pointer to `AGENTS.local.md`; the Setup never creates a `CLAUDE.md`.
_Avoid_: prompt, config, global rules

**Personal values**:
The user's accounts, identities and preferences (GitHub account, Azure user, chat language) that the Repo instructions refer to but do not contain; they always live in `AGENTS.local.md`, never committed.
_Avoid_: secrets, credentials, local config

**Default configuration**:
The skills, plugins and rules the Setup offers, with the default answer to each of its questions, which the user keeps, changes or extends repo by repo. It lives in `SKILL.md`, after the procedure: one entry per skill, plugin or rule, each with the same fields (what to explore, its question, default, what the question shows, prerequisites, install, switch-off, text written, finish notes, how to use).
_Avoid_: catalog, preset, recommended configuration

**Workflow**:
The sequence of steps a new feature goes through, from the design interview to implemented and reviewed tickets; the user starts each step.
_Avoid_: pipeline, feature workflow, process

**Workflow kit**:
The fork of agentic-dev-kit that provides the Workflow steps mattpocock/skills lacks; it holds no rules of its own on git, accounts or trackers, which come from the Repo instructions.
_Avoid_: dev kit, agentic kit

**Prerequisite**:
A skill, plugin or tool that the Repo instructions name and that must be installed for them to work.
_Avoid_: dependency, requirement

**Worker**:
A subagent a skill spawns to do part of its work (implement, review, prove, explore), as opposed to the orchestrator, the session the user talks to.
_Avoid_: delegate, helper

**Worker table**:
The per-session map from each worker role to native or external, with its model and effort; chosen on the launcher's page and injected into that session only. `implementer` is always native, and a role not listed runs as `implementer`.
_Avoid_: role table, routing table

**Native**:
A worker run as the session's own subagent, on the session's account.

**External**:
A worker run as a one-shot headless process on the other provider: prompt in, report out, no resume.
_Avoid_: delegate, headless agent

**Launcher**:
The `fns` command: a local page to pick which CLI and account to start, with the quotas and the Worker table, then it starts that session in the current folder.
_Avoid_: wrapper, router
