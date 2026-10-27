# The Workflow kit is agents-flow, a copy of three agentic-dev-kit skills

Supersedes [ADR 0001](0001-workflow-kit-is-a-fork.md). The Workflow's own steps come from `gianmadd/agents-flow`: `product-review`, `system-architecture` and `program-design`, copied from `salvatorebottiglieri/agentic-dev-kit` (MIT). Implementation goes back to mattpocock's `implement-spec`: it builds the tickets in parallel and its subagents read the sections these steps append to the spec and tickets. `implement-loop`, with its per-ticket reviews and mutation proof, cost more than it was worth, and without it the fork had nothing left worth merging.

## Considered Options

- **Keep the fork, drop `implement-loop` from the install**: keeps upstream merges, but every merge brings back what was removed, for three files that change rarely.
- **Rename the fork**: same merge conflicts.

## Consequences

- Upstream changes to the three skills are not merged; they are ported by hand if wanted.
- The fork `gianmadd/agentic-dev-kit` goes once the repos set up with it have re-run the setup.
