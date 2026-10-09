# Recommended configuration

The line `agents-setup` proposes. Each section is a proposal the user confirms, edits or drops; the indented block under it is what lands in `AGENTS.md`. Placeholders in `<angle brackets>` are filled from the answers. Fork this file to change the recommendation.

## Skills and plugins

### Feature workflow (mattpocock/skills)

- Proposed: yes.
- Prerequisites: skills `setup-matt-pocock-skills`, `grill-with-docs`, `grilling`, `domain-modeling`, `to-spec`, `to-tickets`, `implement-spec`, `tdd`, `code-review`.
- Install: `npx skills add mattpocock/skills -g -y -s <missing skills, space-separated>`.

```md
- New feature: suggest the user runs, one at a time, `/grill-with-docs` → `/to-spec` → `/to-tickets` → `/implement-spec`. These are user-invoked: suggest the command, the user runs it.
```

### Skill nudge

- Proposed: yes, only if the feature workflow is kept.

```md
- Non-trivial task (new feature, multi-file change, refactor) asked outside a skill workflow: ask once whether to use the matching skill instead, naming the command. Small fixes and questions go straight ahead.
```

### Ponytail (DietrichGebert/ponytail)

- Proposed: yes.
- Prerequisite: plugin `ponytail` (`claude plugin list`), or skill `ponytail` (`npx skills ls -g`) for other agents.
- Install, Claude Code: `claude plugin marketplace add DietrichGebert/ponytail && claude plugin install ponytail@ponytail`. Codex: `codex plugin marketplace add DietrichGebert/ponytail && codex plugin add ponytail@ponytail`. Other agents: `npx skills add DietrichGebert/ponytail -g -y`.

```md
- Code: follow ponytail. Agents without it: the smallest complete change, no unrequested abstractions.
```

### Added by the user

For any installed skill or plugin the user adds, one line: when to use it, and the verb. A skill whose `SKILL.md` frontmatter has `disable-model-invocation: true` gets "suggest the user runs `/<name>`"; any other gets "use the `<name>` skill".

## Rules

### Subagents

- Proposed: Claude Code subagents on `sonnet`, effort `high` for implementation and review, `medium` for the rest.

```md
## Subagents (Claude Code)
- `model: sonnet`; effort `high` for implementation and review, `medium` for research and exploration.
```

### Git

- Proposed: GitHub through `gh`, account confirmed by the user; permission level (b).
- Permission levels: (a) read-only; (b) branches and local commits allowed; push, PR, merge to main and ticket changes need explicit confirmation; (c) everything allowed except pushing to main.

```md
## Git
- Run gh as `GH_TOKEN=$(gh auth token -u <account>) gh …`.
- <permission level, as one line>
```

### Azure

- Proposed: only when the user says the repo uses Azure.

```md
## Azure
- Expected: tenant `<tenant>`, subscription `<subscription>`, user `<user>`. Check `az account show` first; on mismatch stop and tell the user, never log in yourself.
- Azure DevOps: always pass `--org <org url> --project <project>`.
- Azure resources (Cosmos, databases, storage…): read-only unless the user explicitly confirms.
```

### Language

- Proposed: everything in English.

```md
## Language
- <chat language> in chat; <code language> for code, comments and commits.
```
