---
name: agents-setup
description: "Set up this repo's agent instructions (AGENTS.md) from a default configuration the user adjusts question by question."
disable-model-invocation: true
---

# Agents setup

Write this repo's **repo instructions**: a rules block telling any coding agent which skills to use when, plus this repo's rules (git, accounts, cloud, subagents, language…), and the user's personal values. Two files: `AGENTS.md` for what belongs to the project, `AGENTS.local.md`, never committed, for what belongs to the user. You ask, the user decides: nothing is written before every question is answered.

What to check, ask, install and write for each skill and rule comes from its entry in section 6, through the fields **Explore**, **Question**, **Default**, **Shown in the question**, **Prerequisites**, **Install**, **Switch off**, **Writes**, **Finish**, **How to use**.

The block is loaded on every turn of every session: rules and the reason behind each, nothing the agent can discover by looking at the repo.

## 1. Explore

Read-only. Gather:

- Agents installed: `which claude codex cursor-agent gemini copilot` (Cursor's CLI is `cursor-agent`).
- Whether the folder is a git repo. If not but its subfolders hold git repos, it is a **workspace** (one folder, several repos, agents launched from the folder): run every repo check in each sub-repo. If neither, it is a **plain folder**: maybe a new project before `git init`.
- `AGENTS.md`, `AGENTS.local.md`, `CLAUDE.md` at the repo root, whether `AGENTS.md` is tracked (`git ls-files AGENTS.md`), and whether any holds an `<!-- agents-repo-setup -->` block. If one does, this is a re-run: its current choices become the defaults in step 2.
- Installed skills and plugins: `npx skills ls -g`, `.agents/skills/` and `.claude/skills/` in the repo, the agents' global skill folders (`~/.claude/skills/`, `~/.codex/skills/`, `~/.agents/skills/`) for skills installed by their own installers (skip hidden folders, `~/.claude/skills/synced/`, which holds claude.ai's own skills, and links into `~/.agents/skills/`, already listed by `npx skills`), `claude plugin list` and `~/.codex/config.toml` for the agents present.
- The **Explore** field of every entry in section 6.

Done when every item above has an answer, "absent" included.

## 2. Ask

Ask every question in one chat message, in plain text (no multiple-choice tool). Open it with what step 1 found, at most five short lines: agents, repo (or workspace) and remote, accounts, existing instruction files, anything else an **Explore** field found. Then each question in this shape:

```md
---

### 3. Skills and plugins

<one or two sentences: what it decides and what changes with each answer, for a user who has never seen this setup>

<what the question needs shown, if anything: a map, a list of entries>

**3a. <the question itself, in a few words>**
- a) <option> **(default)**
- b) <option>
- c) other: tell me

**3b. <second part, only when the question has one>**
- …
```

- A `---` line before every question, a blank line between its parts.
- The options always come last, right under the line that asks, so the user sees what to answer without hunting for it. Anything to read first goes above that line.
- A question with two parts numbers them (**2a.**, **2b.**), each with its own options; a single-part question's bold line has no number, since the heading carries it.
- One option per line, a few words each; keep your own opinion out of it.

End the message with: "Answer like `1 a, 3b none, 6 c`, or 'ok' to keep every default."

The user answers per number, and "ok" takes every default. Follow up on anything unclear until every answer is settled.

- On a re-run, the current choice is the default. When text the user already has outside the markers (step 1's instruction files) answers a question, that is the default, marked with its source: "commits in Italian (from your `CLAUDE.md`)".
- The agents chosen in question 1 shape the others: install commands, switch-offs and per-agent lines cover only those agents.
- Never skip a question because a tool is missing: when an answer needs something not installed, say so in the question; step 3 installs it.

The first two questions decide where the files go:

1. **Agents**: Which agents work in this repo? Options: the installed ones. Default: all installed.
2. **Git and sharing**, up to three parts; number the ones that apply in order (2a, 2b, …). Ask them all in the same message, a later part marked with when it applies ("only if local").
   - Only when the folder is not a git repo: will it be one? Options: yes, `git init` in step 3 (default for a plain folder); no, it stays a workspace (default when sub-repos were found); no, it stays a plain folder. A workspace or a plain folder commits nothing: the later parts apply only if it becomes a repo, so mark them "only if yes"; rules go to `AGENTS.md`, personal values to `AGENTS.local.md`, as in a shared repo.
   - Should the rules be committed for the team (shared: in `AGENTS.md`) or stay yours (local: in `AGENTS.local.md`)? Personal values stay in `AGENTS.local.md` either way. Default: local.
   - Only when local and no `AGENTS.md` exists yet: should the new `AGENTS.md` be committed? It holds the pointer to `AGENTS.local.md` now, and whatever other tools add later (setup skills often write a section there). Default: no.

Then one question per **Question** field in section 6, in its order, numbered from 3. Entries marked as parts of one question become its parts (7a, 7b).

Done when every question that applies is answered.

## 3. Prerequisites

If the user chose `git init` in question 2, run it first. For each kept entry and each agent in use, check the entry's **Prerequisites** against what step 1 found. For each missing one, show its **Install** command and run it only after the user's explicit ok. Re-check after installing. When nothing is missing, say so in one line.

For a default plugin the user dropped but has installed, offer its **Switch off** for this repo, and write it after the user's ok.

Done when every kept entry is installed or knowingly skipped, and every dropped plugin is switched off or knowingly left on.

## 4. Write

Build the blocks from the answers, one block per file, between the markers:

```md
<!-- agents-repo-setup -->
…
<!-- /agents-repo-setup -->
```

The rules block is the header followed by the kept sections, in section 6 order. Where each part goes:

| Rules | `AGENTS.md` block | `AGENTS.local.md` block |
|---|---|---|
| shared | rules block, then the pointer | personal values |
| local | the pointer | rules block, then personal values |

`AGENTS.md`: append the block to an existing file, else create it. A new one is committed in a shared repo, and in a local repo only if the user said so in question 2; otherwise add it to `.git/info/exclude`. In a local repo with a team `AGENTS.md`, the pointer is a committed change to the team's file: tell the user before writing.

`AGENTS.local.md` goes at the repo root and into `.git/info/exclude`. In a workspace or a plain folder both files go in the folder and nothing goes into any `.git/info/exclude`. A settings file you create (a switch-off, the background permissions) in a local repo, or in a shared repo where it is not already tracked, goes into `.git/info/exclude` as well. With the web UI check kept, so does `.playwright-cli/`, in each sub-repo of a workspace: `playwright-cli` writes page snapshots and console logs there.

When a `CLAUDE.md` exists, Claude Code reads it instead of `AGENTS.md` unless it imports `AGENTS.md`. Never create a `CLAUDE.md`; if one exists without the import, offer: (a) `@AGENTS.md` as its first line, a committed change (default in a shared repo); (b) the user sets **Project instructions** to `claude-md-and-agents-md` in Claude Code's `/config`, which applies to all their repos (default in a local repo).

When `AGENTS.md`, `AGENTS.local.md` or `CLAUDE.md` already holds text outside the markers, compare it with the new blocks. For what it repeats, offer to delete it from their text (default: delete; in a team file, a committed change): every repeat is loaded twice in every session. For what contradicts an answer the user gave against their own text, ask how to merge: change their text to match the answer (default), keep both and drop that rule from the block, or rewrite. Never edit their text without an ok; in a team file, an edit is a committed change.

On a re-run, replace only the text between the markers. Before writing, show per file: its path, whether it is committed, and one line per section it gets ("Git: local commits free, push and PRs on confirmation"); on a re-run, the diff instead. Show a full block only when the user asks. Write after their ok. The setup never commits: what should be committed goes in step 5's To do.

## 5. Finish

Report, in this order:

1. **To do**, a numbered list: the files to commit, if any; the **Finish** field of every kept entry; if the rules are local and Codex is in use, that Codex loads them only by following the pointer sentence; if a Codex switch-off was written and the project is not trusted in `~/.codex/config.toml`, that Codex ignores `.codex/config.toml` until they trust it.
2. What was written and where, and anything the user skipped.
3. **How to use it**: the **How to use** line of every kept entry, then one line per skill the user added.

## 6. Entries

Everything the steps above check, ask, install and write, one entry per skill, plugin or rule. Every entry uses the same fields; a missing field means nothing to do:

- **Explore**: what step 1 checks for it, read-only.
- **Question**: the question it adds to step 2, in this section's order, with its options; "part of question N" makes it a part of another entry's question.
- **Default**: the answer "ok" takes.
- **Shown in the question**: what the question shows for this entry before the options.
- **Prerequisites** and **Install**: what must be installed, per agent, and the command; run only after the user's ok.
- **Switch off**: how to turn it off for this repo when the user drops it but has it installed.
- **Writes**: the fenced text written to the rules block, and to `AGENTS.local.md` where marked; `<angle brackets>` are filled from the answers.
- **Finish**: what step 5 tells the user.
- **How to use**: its line in step 5's "How to use it".

An entry is **kept** when the user keeps it (skills) or answers so that it writes something (rules).

### Fixed text

#### Pointer

Always written to `AGENTS.md`, inside the markers, so every agent also loads `AGENTS.local.md`: Claude Code through the `@` import, other agents by following the sentence. In a workspace or a plain folder, leave out the last sentence: there are no worktrees.

```md
## Personal instructions

Before your first reply or action, read `AGENTS.local.md` next to this file, if it exists, and follow it as part of these instructions: @AGENTS.local.md
It holds this checkout's personal rules; it is not committed and not every agent loads it on its own. In a worktree, read it from the main checkout root, `$(git rev-parse --path-format=absolute --git-common-dir)/..`.
```

#### Header

Always written, first in the rules block. The second sentence only when `AGENTS.local.md` is a separate file from the rules (shared repo, workspace, plain folder).

```md
# Agent instructions

Standing rules for every agent working here. Personal values are in `AGENTS.local.md`.
```

#### Personal values

The `AGENTS.local.md` lines of every entry, as a bullet list. When the rules share that file (local repo), they come after the rules under a `## Personal values` heading; otherwise the list stands alone.

### Skills and plugins

- **Question**: Skills and plugins, two parts. (a) Which of the default skills and plugins should this repo use? Show, under "What the defaults give", the **Shown in the question** text of every entry below; then list the entries, each marked installed or missing per agent. Options: all (default); drop some: tell me which; none. An entry that works only with another ("only together with") goes when that one goes. (b) Do you want to add others? First list the installed skills and plugins outside the defaults grouped by source, one line per group, names only: one group per skill package in `npx skills ls -g` (e.g. mattpocock/skills), one per agent's plugins, one for the repo's own skills. Leave out this setup itself, the skills the default entries need as prerequisites, disabled plugins, plugins synced from claude.ai, and plugins an agent bundles on its own (Codex's `openai-*`). The user can ask what any name does. For each one added, ask when it is used and whether the user starts it. Default: none.

#### Workflow (mattpocock/skills and gianmadd/agentic-dev-kit)

- **Explore**: which of the Prerequisites are installed globally (`npx skills ls -g`) and which in the repo (`npx skills ls`).
- **Default**: on.
- **Shown in the question**:
  ```md
  New features, one step at a time; you start each step, the agent suggests the next and says which optional step it would skip:
  1. `grill-with-docs`: interviews you until the design is settled
  2. `to-spec`: writes the spec on the issue tracker
  3. `product-review`: an HTML mockup of the UI (only with the web UI check)
  4. `system-architecture`: diagrams, contracts and domain invariants (skipped for small changes)
  5. `to-tickets`: splits the spec into tickets
  6. `program-design`: the signatures and files each ticket will touch
  7. `implement-loop`: builds the tickets with TDD, reviews each on three axes, proves the tests with mutations, checks test, lint and UI; asks before push and PR as your git rules say
  The skills install in this repo, not globally.
  ```
- **Prerequisites**: from mattpocock/skills, `setup-matt-pocock-skills`, `grill-with-docs`, `grilling`, `domain-modeling`, `to-spec`, `to-tickets`, `tdd`, `code-review`; from gianmadd/agentic-dev-kit, `system-architecture`, `program-design`, `implement-loop`, and `product-review` with the web UI check.
- **Install**, into the repo (into the folder in a workspace), never with `-g`; `<agents>` are the agents in use (`claude-code`, `codex`, …):
  - mattpocock/skills: `npx skills add mattpocock/skills -y -a <agents> -s <skills>`. In a shared repo, every one of them, so the team gets them; in a local repo, only those not installed globally: a global skill wins over the repo's copy in Claude Code, and Codex would list both.
  - `npx skills add gianmadd/agentic-dev-kit -y -a <agents> -s <skills>`, always.

  They land in `.agents/skills/`, with links in `.claude/skills/`, and `npx skills` records them in `skills-lock.json`.
- **Writes** (step 3 only with the web UI check, the steps renumbered without it; the last paragraph only when Ponytail is dropped):
  ```md
  ## Workflow

  New features go through the Workflow, one step at a time. The user starts each step: when one finishes, suggest the next instead of running it yourself, and say which optional step you would skip and why. The steps and `setup-matt-pocock-skills` are commands only the user can run, so they are missing from your skill list even when installed: suggest them by command (`/grill-with-docs` in Claude Code, `$grill-with-docs` in Codex), never as missing.

  1. `grill-with-docs`: interview the user until the design is settled; updates `GLOSSARY.md` and ADRs. Before the spec, have a subagent try to disprove what the conversation assumed about the current code: a premise that falls changes the design.
  2. `to-spec`: turn the settled design into a spec on the issue tracker.
  3. `product-review`, when the feature touches UI: an HTML mockup with success criteria per persona and edge case.
  4. `system-architecture`, unless it is a bug fix or a trivial feature: diagrams, contracts and domain invariants, appended to the spec.
  5. `to-tickets`: split the spec into tickets with blocking relations.
  6. `program-design`, unless no signature changes (config, data, mechanical rename): call tree, file changes and signatures, appended to each ticket.
  7. `implement-loop`: implement the tickets with subagents and TDD, review each on three axes, prove the tests with mutations, then the test, lint and UI gates.

  If `docs/agents/` is missing, the issue tracker is not configured yet: suggest `setup-matt-pocock-skills` first.

  Ponytail is not installed here: `implement-loop` runs without its ponytail axis.
  ```
- **Finish**: if `docs/agents/` is missing, run `setup-matt-pocock-skills` now. It writes its section to `AGENTS.md` (to `CLAUDE.md` if one exists), so that section is tracked exactly when that file is. Its new files (`docs/agents/`, `GLOSSARY.md`, `docs/adr/`) start untracked: commit them or add them to `.git/info/exclude`. The same for `.scratch/`, where a local tracker keeps its issues and `product-review` its mockups. The installed skills (`.agents/skills/`, `.claude/skills/`, `skills-lock.json`): in a shared repo, commit them; in a local repo, add them to `.git/info/exclude`. Add `.claude/worktrees/` to `.git/info/exclude` in both: `implement-loop` works there. In a local repo with global mattpocock skills, say that once no other repo uses those, they can go, and that repos set up before still need them until their setup is re-run.
- **How to use**: New feature: start `grill-with-docs`; when each step ends, the agent suggests the next. `implement-loop` asks before push and PR as the git rules say. Update the skills with `npx skills update -p`.

#### Background permissions

- **Explore**: `.claude/settings.local.json`, if it exists, and whether it is tracked; the repo's test, build and run commands (`package.json` scripts, `Makefile`, `*.csproj`, Gradle).
- **Default**: on, only together with the Workflow, and only for Claude Code.
- **Shown in the question**: `implement-loop` runs its subagents in the background, where Claude Code cannot ask you for permission: a command not pre-approved is refused and the ticket stalls. This pre-approves what they need, and always asks for push and destructive git.
- **Writes**, to `.claude/settings.local.json`, not to the rules block: merge into `permissions`, keeping what is there.
  ```json
  {
    "permissions": {
      "allow": ["Edit", "Bash(git status*)", "Bash(git diff*)", "Bash(git log*)", "Bash(git show*)", "Bash(git rev-parse*)", "Bash(git branch)", "Bash(git branch --show-current)", "Bash(git switch*)", "Bash(git checkout -b*)", "Bash(git add*)", "Bash(git commit*)", "Bash(git merge*)", "Bash(git worktree*)", "Bash(<each test, build and run command>)", "Bash(ls *)", "Bash(cat *)", "Bash(head *)", "Bash(tail *)", "Bash(grep *)", "Bash(find *)", "Bash(wc *)", "Bash(pwd)", "Bash(lsof *)", "Bash(kill *)"],
      "ask": ["Bash(git push*)", "Bash(git reset --hard*)", "Bash(git clean*)", "Bash(git branch -D*)", "Bash(git rebase*)", "Bash(git switch -f*)", "Bash(git switch --discard-changes*)", "Bash(git checkout -- *)", "Bash(git checkout .*)", "Bash(git restore*)", "Bash(git stash drop*)", "Bash(git commit --amend*)"]
    }
  }
  ```
  The run commands come from the repo (e.g. `Bash(npm *)`, `Bash(node *)`, `Bash(dotnet *)`, `Bash(./gradlew *)`): for `package.json` scripts, both the package manager that runs them (`npm`, `pnpm`, `yarn`) and the tools they call, since agents run `npm test`, not `node --test`; add `Bash(playwright-cli *)` with the web UI check, `Bash(maestro *)` with Android or iOS, `Bash(adb *)` with Android, `Bash(xcrun simctl *)` with iOS, and `Bash(peekaboo *)`, `Bash(xcodebuild *)` with macOS. `ask` wins over `allow`, and any git command not listed asks too. The git list is what the subagents need: reading, branches, commits, merges into their own branches, worktrees. With git level (a), keep only the reading commands (`status`, `diff`, `log`, `show`, `rev-parse`, `branch`), and warn in the question that `implement-loop`'s subagents then cannot commit, so its tickets stall; with level (c), also allow `Bash(git push*)` and ask only for `git push --force*`, `git push -f*` and `git push * main*` among the pushes: that catches only an explicit push to main, not a plain `git push` from main, which is left to the rule in `AGENTS.md`; with level (d), replace the git commands with `Bash(git *)` and drop `ask`. With the `GH_TOKEN=` prefix of the GitHub account rule, push, pull and fetch still ask at (c) and (d): an `allow` rule does not match past that prefix, an `ask` rule does.
- **How to use**: `implement-loop`'s background agents run without stopping on permission prompts; the git commands not pre-approved still ask you.

#### Skill nudge

- **Default**: on, only together with the Workflow.
- **Shown in the question**: For a larger task outside the Workflow, the agent asks once whether to use it.
- **Writes**, at the end of the Workflow section:
  ```md
  When the user asks for a non-trivial task (new feature, multi-file change, refactor) outside the Workflow, ask once whether to go through it instead. Small fixes and questions go straight ahead.
  ```
- **How to use**: Small fixes and questions: just ask; for a larger task the agent asks once whether to use the Workflow.

#### Ponytail (DietrichGebert/ponytail)

- **Default**: on.
- **Shown in the question**: Ponytail is always on: the agent writes the least code that fully solves the task. With the Workflow, `implement-loop` also reviews each ticket on a ponytail axis.
- **Prerequisites**: Claude Code plugin `ponytail@ponytail` (`claude plugin list`); Codex plugin `ponytail@ponytail` (`[plugins."ponytail@ponytail"]` in `~/.codex/config.toml`); other agents, skill `ponytail` (`npx skills ls -g`).
- **Install**: Claude Code `claude plugin marketplace add DietrichGebert/ponytail && claude plugin install ponytail@ponytail`; Codex `codex plugin marketplace add DietrichGebert/ponytail && codex plugin add ponytail@ponytail`; other agents `npx skills add DietrichGebert/ponytail -g -y`.
- **Switch off**: Claude Code, `"enabledPlugins": {"ponytail@ponytail": false}` in `.claude/settings.local.json`; Codex, `[plugins."ponytail@ponytail"]` with `enabled = false` in `.codex/config.toml` (read only once the project is trusted in Codex).
- **Writes**:
  ```md
  ## Code

  Write and review code the ponytail way: the plugin loads it every session; where ponytail is installed as a skill instead, load the `ponytail` skill before writing code.

  - Reviews outside the Workflow: `ponytail-review`.
  ```
  Without the Workflow, the last line reads "Reviews: `ponytail-review`."
- **How to use**: Ponytail is always on: no command needed.

#### Added by the user

- **Writes**: one line per skill or plugin: when to use it, by name; "suggest `<name>`" if the user starts it, else "use `<name>`". It goes at the end of the Workflow section, or in a `## Skills` section of its own when the Workflow is dropped.
- **How to use**: one line per skill: when to use it.

### Rules

#### UI check

- **Explore**: what UI the repo has. Web: a frontend framework in `package.json` (React, Vue, Svelte, Angular, Next…), or `.html`, `.razor`, `.cshtml` files outside dependencies. Android: an `AndroidManifest.xml`, or the `com.android.application` plugin in a Gradle file. iOS: an `.xcodeproj` or `.xcworkspace` with an iOS target. macOS: an Xcode project with a macOS target (`SDKROOT = macosx` in `project.pbxproj`), or a `Package.swift` with a macOS executable.
- **Question**: always asked, whatever was found. How should the agent check UI changes before calling them done? Say what was found and where ("Android app in `app/`"). Options, any combination: web, with `playwright-cli`; Android, with Maestro; iOS, with Maestro (on a Mac); native macOS apps, with Peekaboo; or none. Say what each needs installed, so the user picks only what they will use.
- **Default**: what was found (web, Android, iOS, macOS, or several); none when the repo has code but no UI; web in a new or empty repo.
- **Prerequisites**: web, command `playwright-cli` (Node 18+), skill `playwright-cli` (`npx skills ls -g`), and a browser: Google Chrome, which `playwright-cli` uses by default, or Playwright's Chromium (`chromium-*` in Playwright's browser cache, `~/Library/Caches/ms-playwright` on macOS). Android and iOS: command `maestro` and Java 17+ (`java -version`). Android also `adb` (Android platform-tools, usually there with Android Studio): the agent installs the app and reads crashes through it. iOS also Xcode and its Command Line Tools (`xcode-select -p`). macOS: command `peekaboo` (macOS 15+), Xcode and its Command Line Tools, and Screen Recording and Accessibility permissions for the app the agent runs in (`peekaboo permissions status`): granting them in System Settings is the user's job. Install only what the chosen platforms need.
- **Install** (every agent): web, `npm install -g @playwright/cli@latest`, then `npx skills add microsoft/playwright-cli -g -y -s playwright-cli`; without Google Chrome, `playwright-cli install-browser chromium` too, and offer `export PLAYWRIGHT_MCP_BROWSER=chromium` in the user's shell profile so it becomes the default. Android and iOS: `brew tap mobile-dev-inc/tap && brew trust --formula mobile-dev-inc/tap/maestro && brew install mobile-dev-inc/tap/maestro` (other systems: `curl -fsSL "https://get.maestro.mobile.dev" | bash`); Java 17+ if missing, e.g. `brew install openjdk@17`. Android: `brew install --cask android-platform-tools` (other systems: https://developer.android.com/tools/releases/platform-tools). iOS: Xcode from the App Store, then `xcode-select --install`. macOS: `brew install openclaw/tap/peekaboo`, Xcode as for iOS, then tell the user to grant the two permissions.
- **Writes**, one line per chosen platform:
  ```md
  ## UI check

  Before calling a UI change done, check it running: passing tests don't show a blank screen or a hidden button. When done, stop the processes you started: a server left running holds its port next time.

  - Web: start the app (find how from the repo), open it with `playwright-cli`, walk through the changed flow and check the console for errors. If Google Chrome is missing, pass `--browser=chromium`.
  - Android: install the app on a running emulator or device (`./gradlew installDebug`, or how the repo builds), walk through the changed flow with Maestro (`maestro hierarchy` to find elements, a short flow YAML run with `maestro test <flow>.yaml`) and check `adb logcat` for crashes. With no device running, start an emulator if one is configured (`maestro start-device`), else ask the user.
  - iOS: build the app for the simulator and install it on a booted one (`xcrun simctl install booted <app>`, or how the repo builds), walk through the changed flow with Maestro as above, and check the simulator log for crashes. With no simulator booted, boot one (`maestro start-device`), else ask the user.
  - macOS: build and launch the app (`xcodebuild`, or how the repo builds; then `open <app>`), walk through the changed flow with Peekaboo (`peekaboo see --app <name> --json` to find elements, then `peekaboo click` and `peekaboo type`), and check for crash reports in `~/Library/Logs/DiagnosticReports`. Quit the app when done. If Peekaboo lacks permissions, ask the user to grant them.
  ```
- **How to use**: UI changes: the agent checks them running, in the browser, on the emulator, on the simulator or on the Mac, before saying they're done.

#### Subagents

- **Question**: Which model should the coordinator (the main session that spawns subagents) use, and which model and effort should subagents use, per agent? Options: the default; other: tell me.
- **Default**, per agent in use: coordinator Claude Code `opus`; subagents Claude Code `sonnet`, Codex `gpt-6-luna`, effort `medium`.
- **Writes**, one line per agent in use:
  ```md
  ## Subagents

  Subagents use the model and effort below: pass both explicitly at every spawn.

  - Claude Code: `model: sonnet`; effort `medium`.
  - Codex: model `gpt-6-luna`; reasoning effort `medium`.
  ```
  With Claude Code, also merge `"model": "opus"` into `.claude/settings.local.json`, keeping what is there: a rule cannot change the session's own model.

#### Workers launcher

- **Explore**: whether a shell rc file (`~/.zshrc`, `~/.bashrc`) already sources `workers/workers.sh` from this skill's folder; `which python3`.
- **Question**: only when Claude Code and Codex are both in use and the launcher is not installed. Install the `--workers` launcher? It is for this user, not this repo: it goes in their shell rc file and works in every repo. Options: yes; no.
- **Default**: no.
- **Shown in the question**:
  ```md
  `claude --workers` or `codex --workers` (aliases like `claude-work` too) first opens a local page with each subscription's remaining quota, where you pick, per worker role (implementer, reviewer, prover, explorer), whether the session's own subagents run it (native) or a one-shot headless run on the other CLI (external), with its model and effort; `implementer` is always native. The table is injected into that session only; without the flag, nothing changes. External workers run with permissions bypassed, so they are meant for roles that only return a report.
  ```
- **Prerequisites**: `python3`, `claude`, `codex`.
- **Install**: append `source <this skill's folder>/workers/workers.sh` to the rc file of the user's shell (`$SHELL`), with the folder's absolute path.
- **Finish**: open a new terminal, or `source` the rc file, before the first `--workers`.
- **How to use**: `claude --workers` / `codex --workers`: pick who runs each worker role for this session, based on the quotas the page shows.

#### GitHub account

- **Explore**: `git remote -v` (host: GitHub or not; protocol: HTTPS or SSH), `git config --get-all credential.helper` and `git config --get-all credential.https://github.com.helper`, `git config user.email`, `gh auth status` (the logged-in accounts).
- **Question**: always asked. Which GitHub account does this repo use?
  - Options: the accounts in `gh auth status`, then "not on GitHub". If `gh` is missing, say a GitHub account needs it.
  - With only one account, ask the user to confirm it and say no prefix rule is written.
  - Git's credential helper for GitHub: `credential.https://github.com.helper` wins over `credential.helper`; skip empty entries (they reset the list). It is `gh` when the value ends in `gh auth git-credential`, whatever path or `!` precedes it.
  - In a workspace whose sub-repos belong to different owners, offer one option per account plus "one account per sub-repo"; that option's default gives each sub-repo its owner's account when `gh` holds it.
  - When the chosen account and `git config user.email` disagree, warn the user; git config is theirs to change. When the remote is SSH or git's credential helper is not `gh auth git-credential`, say the rule covers only `gh`: git authenticates with their SSH key or helper.
- **Default**: with the remote on GitHub, the account matching `git config user.email` (a GitHub noreply email carries the account name), else the active one; with the remote elsewhere, "not on GitHub"; with no remote yet, the account matching `git config user.email` if any, else "not on GitHub". The rule is written only when the user chose an account and `gh` holds more than one: with a single account there is nothing to mix up.
- **Prerequisites**: `gh`.
- **Install**: `brew install gh` (other systems: https://cli.github.com); then the user runs `gh auth login`: logging in is their job.
- **Writes**, appended to the Git section, after any workspace sentence:
  ```md
  This machine's active GitHub account, which may not be this repo's, is used by `gh`<" and by git through its `gh` credential helper">. Prefix every `gh` command<", and every `git` command that reaches GitHub (push, pull, fetch),"> with `GH_TOKEN=$(gh auth token -u <account>)`, `<account>` from `AGENTS.local.md`; if it is missing, ask the user.
  ```
  `<account>` stays as written: the value lives in `AGENTS.local.md`. The quoted parts only when the remote is HTTPS (or there is no remote yet) and git's credential helper is `gh auth git-credential`: `GH_TOKEN` reaches git only through that helper. In a workspace, the first sentence says "the sub-repo's" instead of "this repo's"; with different accounts, `<account>` is the sub-repo's account in `AGENTS.local.md`. When only some sub-repos qualify for the quoted parts, keep them and add "`<sub-repo>` uses SSH: there the prefix covers only `gh`." (one sentence per sub-repo).

  `AGENTS.local.md`:
  ```md
  - GitHub account: `<account>`
  ```
  In a workspace with different accounts: ``- GitHub account: `<account>` for `<sub-repo>`, …``.

#### Git permissions

- **Question**: Git, two parts; this is the first. What may the agent do with git and GitHub without asking?
- **Default**: (b).
- **Options**: (a) read-only: anything that writes to git or GitHub needs confirmation; (b) branches and local commits allowed; push, PRs, merge to main and issue changes need confirmation; (c) everything allowed except pushing to main and merging PRs into main; (d) everything allowed, with no confirmation.
- **Writes**, not in a plain folder (no git):
  ```md
  ## Git

  - Allowed without asking: <level>.
  - Ask first: <level>.
  - Always ask first, whatever the above allows: `push --force`, `reset --hard`, `branch -D`, `clean`, and rewriting pushed commits. They lose work.
  ```
  The two lines per level: (a) nothing / any command that writes to git or GitHub; (b) creating and switching branches, local commits / push, opening or merging PRs, merging to main, creating or editing issues; (c) branches, commits, push to branches other than main, opening and merging PRs into other branches, creating or editing issues / pushing to main, merging PRs into main; (d) everything, main and destructive commands included / nothing. With (d), only the first line; the third line not with (a): it already asks for everything.
  In a workspace, appended:
  ```md
  This folder is not a git repo: run git and `gh` inside the sub-repo concerned (`git -C <repo> …`). Sub-repos: <list>.
  ```

#### Git conventions

- **Explore**: `git branch -a` and `git log --oneline -20`: the branch names and commit messages already in use.
- **Question**: part of the Git question. How should the agent name branches and write commits and PRs? Options: the default; the convention the history already follows, when it differs (named in one line, e.g. "Conventional Commits"); other: tell me.
- **Default**: as below.
- **Writes**, not in a plain folder, as bullets in the Git section after the permission lines:
  ```md
  - Branches: `<type>/<task>-<short-title>`, e.g. `feat/420-login-page`; type `feat`, `fix`, `refactor`, `docs` or `chore`; no `<task>` when the work has no task.
  - Commit messages and PR descriptions: say what changed and why, in plain words. Short and explanatory, not a wall of technical detail.
  ```
  With the history's convention, write it in the same shape instead.

#### Secrets

Always written, no question: a leaked key is the one mistake that cannot be taken back.

- **Writes**:
  ```md
  ## Secrets

  Never print, commit or paste secrets: `.env` files, keys, tokens, passwords, connection strings. To check one is set, test that it exists, not its value.
  ```

#### Azure resources

- **Explore**: `az account show`, if `az` is installed.
- **Question**: Azure, two independent parts; this is the first. Does the code run on or read Azure resources? If yes: tenant, subscription and user. If `az` is missing, say a yes needs the Azure CLI.
- **Default**: no; if yes, the values from `az account show`.
- **Prerequisites**: Azure CLI (`az`).
- **Install**: see https://learn.microsoft.com/cli/azure/install-azure-cli (macOS: `brew install azure-cli`).
- **Writes**:
  ```md
  ## Azure resources

  - They live in tenant `<tenant>`, subscription `<subscription>`. Before any `az` command that touches them (anything but `az devops`, `az boards`, `az repos`, `az pipelines`), check `az account show` against these and the user in `AGENTS.local.md`. On a mismatch stop and tell the user to run `az login --tenant <tenant>`: logging in is their job.
  - Read-only: any write needs the user's explicit confirmation, whatever the git permissions say.
  ```
  `AGENTS.local.md`:
  ```md
  - Azure user: `<user>`
  ```

#### Azure DevOps

- **Explore**: `az devops configure --list`, if `az` is installed.
- **Question**: part of the Azure question. Do the work items live on Azure DevOps? If yes: org and project. If `az` is missing, say a yes needs the Azure CLI.
- **Default**: yes when the remote is on `dev.azure.com` (org and project from it), else no; if yes, the values from the remote or `az devops configure --list`. Independent of Azure resources: the board can sit in another tenant, and `az devops` does not follow the active subscription.
- **Prerequisites**: Azure CLI with the `azure-devops` extension (`az extension list`).
- **Install**: the Azure CLI as above, then `az extension add --name azure-devops`.
- **Writes**:
  ```md
  ## Azure DevOps

  - Work items live in org `<org url>`, project `<project>`; a bare number like "task 420" is a work item there. Use the `azure-devops` MCP server if available, else pass `--org` and `--project` on every `az devops`, `az boards`, `az repos` and `az pipelines` call.
  - Reading is free; creating or editing work items needs the user's explicit confirmation.
  ```
  In the Git section, say "work items" instead of "issues".
- **Finish**: with the Workflow kept, pick "Other" as issue tracker in `setup-matt-pocock-skills` and describe Azure DevOps.

#### Language

- **Question**: two parts. Which language for chat? Which for code, comments and commits?
- **Default**: English for chat, and for code, comments and commits. The chat language is personal: it goes to `AGENTS.local.md`, so a shared repo does not impose it on the team.
- **Writes**:
  ```md
  ## Language

  Code, comments and commits in <language>.
  ```
  When the languages differ, say which goes where: "Code and comments in English; commits in Italian."
  `AGENTS.local.md`:
  ```md
  - Chat language: <language>
  ```
