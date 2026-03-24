# Delegation Boundaries

The runtime skill is a controller, not a worker.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- read repo-local retry contract docs when present
- create the round branch and worktree
- update `orchestrator/state.json`
- record artifact paths, retry-state fields, and stage markers exactly as the repo-local contract requires
- perform squash-merge bookkeeping after approval

## The Orchestrator Must Delegate

- task selection
- roadmap edits
- round planning
- implementation
- review decisions
- merge-note authoring

## Subagent Rules

- Use real subagents, not simulated roles.
- Prefer repo-local `.codex/agents/orchestrator-<role>.toml` definitions when they exist.
- If the matching repo-local agent file is missing, fall back to `orchestrator/roles/<role>.md`.
- Use a fresh subagent for each stage.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the subagent to finish before continuing.
- If a required repo-local role definition is missing from both locations, stop instead of inventing one.

## If Real Subagents Are Unavailable

Stop and tell the user the runtime skill cannot honor its delegation contract in the current environment.
