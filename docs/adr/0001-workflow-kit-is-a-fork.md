# The Workflow kit is a fork of agentic-dev-kit

Superseded by [ADR 0002](0002-workflow-kit-is-agents-flow.md).

The Workflow comes from `gianmadd/agentic-dev-kit`, a GitHub fork of `salvatorebottiglieri/agentic-dev-kit`, not from the original. The original sets its own rules on git, trackers and session-end checks, and expects to live in `~/agentic-dev-kit`; the Setup needs those rules to come from the Repo instructions and the skills to install per repo. The fork changes only the `SKILL.md` files and adds `implement-loop/criteria.md`, leaving every other file untouched, so the original's updates come in with a plain `git merge`.

## Considered Options

- **Use the original and override it from `AGENTS.md`**: no maintenance, but two sources that contradict each other on every run of `implement-loop`.
- **Propose the changes upstream**: the cleanest end state, deliberately not pursued; the fork keeps control of when changes land.
- **Copy the skills into this repo**: simplest, but loses the original's updates.

## Consequences

- Every update of the original costs a merge in the fork; if the review criteria in its `WORKFLOW.md` change, `criteria.md` is copied again.
- The fork is never pinned to a tag: the merge is the version gate.
