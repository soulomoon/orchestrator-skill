# ADR-0012: Reuse same-role subagents across rounds

## Status

Accepted

## Context

The delegation contract previously treated a merged round as an example point
where a host subagent handle could be closed. That was safe but wasteful: the
role identity often remains useful after a round completes. A planner can plan
the next round, an implementer can implement the next round, and a reviewer can
review the next round, as long as the controller supplies the new assignment
and the subagent is not carrying unresolved stale state.

The real safety boundary is role compatibility, not round completion.

## Decision

Do not close or release a finished subagent merely because a round merged.

Keep two compatibility concepts:

- same-stage resume compatibility: same role, same round or roadmap-update id,
  same branch/worktree, same selected lineage, and no instruction conflict;
- cross-round same-role reuse compatibility: same runtime role in the same
  repository control plane, finished or idle handle, no unresolved prior task,
  no stale or unusable recovery mark, and an explicit new assignment that
  rebinds the handle to the new round id, branch/worktree, lineage, and
  artifacts while telling it to reload current state and treat previous round
  details as historical context only.

The controller should prefer a reusable same-role handle before spawning a new
subagent for a later round. Role boundaries still require separate handles: a
reviewer must not implement, and an implementer must not review.

Close or release a handle only when it is finished and no longer same-role
reusable, such as after terminal control-plane completion, an incompatible
role prompt or control-plane migration, a role boundary, a stale or unusable
recovery mark, or a host resource constraint.

## Consequences

**Positive:**
- Role agents can retain useful role-specific context across a roadmap family.
- The controller avoids unnecessary subagent churn between rounds.
- Same-round retry safety remains stricter than cross-round role reuse.

**Negative:**
- Reused agents must receive explicit reset/rebind instructions for every new
  round to avoid carrying old round assumptions forward.

**Neutral:**
- The controller still owns state and artifact routing.
- Subagents still do not cross role boundaries.
