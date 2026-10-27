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
`gianmadd/agents-flow`, the planning steps mattpocock/skills lacks, copied from agentic-dev-kit; it holds no rules of its own on git, accounts or trackers, which come from the Repo instructions.
_Avoid_: dev kit, agentic kit

**Prerequisite**:
A skill, plugin or tool that the Repo instructions name and that must be installed for them to work.
_Avoid_: dependency, requirement
