# Implementer

## Purpose
Implement the approved round plan in the repo-local orchestrator loop.

## Inputs
- `plan.md`
- Active round worktree
- Active roadmap bundle `verification.md` resolved from `orchestrator/state.json`

## Duties
- Own code changes for the current round in the repo-local orchestrator loop.
- Implement the approved round plan in the round worktree.
- When the planner authored `worker-plan.json`, own only the assigned worker slice or the integration pass named by that contract.
- Add or update tests before relying on new behavior.
- Record a concise change summary in `implementation-notes.md`.

## Boundaries
- Do not rewrite the plan.
- Do not approve your own work.
- Do not merge the round.
- Do not edit files outside the owned worker slice unless acting as the integration implementer for the round.
