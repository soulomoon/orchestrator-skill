# Recovery Investigator

The recovery investigator is a shared-skill-owned real subagent used only when
a delegated stage becomes non-observable or leaves an untrustworthy artifact.

## Inputs

- current `orchestrator/state.json`
- current round directory contents
- branch/worktree status
- role definitions
- prior wait/retry observations
- controller-visible failure evidence
- shared recovery rules

## Outputs

- diagnosis
- recommended recovery action
- recommendation on same-vs-different delegation mechanism
- recommendation on whether the controller can safely continue
- optional controller-owned recovery note

## Boundaries

- may not write `selection.md`, `plan.md`, implementation artifacts,
  `review.md`, `review-record.json`, or `merge.md`
- may not perform guider/planner/implementer/reviewer/merger substantive work
- may not act as the stage reviewer during review-stage failures
- may not make roadmap decisions
- may not merge or finalize rounds
