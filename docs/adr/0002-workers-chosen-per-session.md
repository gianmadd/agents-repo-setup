# Workers are routed per session, through a `--workers` launcher

Agents Hub was retired (gianmadd/agents-hub ADR 0009) and its routing moved here: when a skill spawns a worker, it runs either as a native subagent or as a headless run on another subscription. The user decides who does what at each launch, from each subscription's remaining quota, so the Worker table cannot live in the Repo instructions: it is chosen when the session starts and injected into that session only (`--append-system-prompt` for Claude, `-c developer_instructions` for Codex). `claude --workers` or `codex --workers` opens a local page with the quotas and the table, then starts the CLI. A role is **native** (the session's own provider and account) or **external** (the other provider), with its model and effort, so one table shape serves both; `implementer` is always native, since the orchestrator works with it and unlisted roles fall back to it. External workers are one-shot, prompt in and report out, with no resume, and are meant for the roles that only return a report; that is also why bypassing their permissions is acceptable.

This brings back, in a reduced form, the Hub's original purpose: one place to start sessions and pick which subscription does what. That is acceptable this time because what is left is only that choice. There is no app, no per-prompt router and no surfaces to watch, only shell functions and a page that lives until you confirm it.

## Considered options

- **The table in `AGENTS.local.md`, per repo**: rejected. Quotas belong to the subscription and change during the day, so the user would edit every repo.
- **One global table file**: rejected. Two sessions in two repos could not run different tables at once.
- **A picker in the terminal instead of a page**: simpler, but the user preferred a page. A `file://` page cannot send the choice back, so the page is served by a one-shot Python stdlib server.
- **Resuming external workers**: dropped. `implement-loop` re-spawns a reviewer instead of following up, so the contract stays prompt in, report out.
- **agy as a provider or with its own table**: dropped from the project.

## Consequences

- The Repo instructions don't change. The Subagents entry still sets native models when a session starts without `--workers`; the Setup only asks whether to install the launcher.
- Claude records the system prompt on the first request and reuses it on resume (`--system-prompt-snapshot`), so a resumed conversation keeps the table it started with; resume is not handled otherwise.
- Claude's quota is read from the text of `claude -p /usage`, which is undocumented and may change with the CLI. Codex's comes from `codex app-server`, documented but marked experimental.
- The last confirmed table is saved per repo and per provider in `.git/agents-workers.json`, shared by its worktrees and never committed.
