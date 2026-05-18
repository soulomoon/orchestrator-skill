# Delegation Boundaries

The runtime skill is a controller, not a worker.
Runtime role loading happens only from `orchestrator/roles/`.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- read `orchestrator/artifact-manifest.md` before resolving artifact paths
- read `orchestrator/active-roadmap-bundle.md` before interpreting the active
  roadmap bundle
- read the active roadmap bundle and any roadmap-specific retry overrides in
  `verification.md`
- create round branches and round worktrees
- create worker branches and worker worktrees when `round-plan-record.json`
  requires them
- create roadmap-update branches and worktrees after successful round merges
  when semantic roadmap updates are required, or after valid plan-stage
  `roadmap-update-request.md` when a pre-implementation split is required
- update `orchestrator/state.json`, including active roadmap metadata when a
  reviewed and approved `update-roadmap` stage lawfully activates a new revision
- apply reviewer-approved status-only round closeout directly in the canonical
  round worktree before merge, limited to the roadmap-view selectors, compact
  completion pointers, and compact history entries recorded in
  `review-record.json`
- write and revalidate controller-owned `closeout-record.json` for status-only
  round closeout
- derive merge admissibility and perform squash merge bookkeeping during
  `finalize-round`
- serialize semantic roadmap updates through the single
  `state.json.roadmap_update` record
- preserve an existing planner-authored `roadmap-update-request.md` verbatim in
  the roadmap-update worktree for a planner-requested update, without
  authoring or editing its substance
- record retry-state fields and stage markers exactly as the repo-local
  contract requires
- consume rejected-review retry targets from `review-record.json` and
  redispatch the owning role without authoring the required changes directly
- resolve artifact paths exactly as `orchestrator/artifact-manifest.md`
  prescribes
- launch and use the repo-local `recovery-investigator` from
  `orchestrator/roles/recovery-investigator.md` for recovery diagnosis when
  controller-visible evidence for the active stage is missing or untrustworthy,
  and run controller-owned recovery repair actions directly without authoring
  the investigation itself
- re-read the recorded canonical round worktree and treat it as the primary
  observation surface for round-owned stage artifacts during recovery
- clear stale blockage bookkeeping and recreate missing worktrees when those
  controller-owned repairs can recover an already-produced stage result without
  authoring new stage content
- attempt `recovery-investigator` for non-terminal delegated-stage stop
  situations before recording delegation blockage
- record the precise blockage in
  `orchestrator/state.json.resume_errors.controller` only after an attempted
  `recovery-investigator` and the full recovery ladder fail to produce a
  qualifying recovery path, or when no qualifying
  `recovery-investigator` can launch through any available delegation
  mechanism
- perform squash-merge bookkeeping after approval

## The Orchestrator Must Delegate

- normal round task selection, round planning, worker fan-out assignment
  design, and semantic roadmap bundle edits
- roadmap update review
- implementation
- integration implementation when worker fan-out is active
- review decisions
- all substantive planner/implementer/reviewer outputs, including
  retry-stage outputs

## Subagent Rules

- Use real subagents, not simulated roles.
- Load each runtime role only from `orchestrator/roles/<role>.md`.
- Prefer reusing or resuming an existing compatible subagent before spawning a
  fresh one. Compatibility has two levels:
  - Same-stage resume compatibility requires the same runtime role, same round
    id or roadmap-update id, same branch/worktree, same selected lineage, and
    no unresolved instruction conflict with the current stage.
  - Cross-round same-role reuse compatibility requires the same runtime role in
    the same repository control plane, a finished or idle handle, no unresolved
    prior task, no stale or unusable recovery mark, and an explicit new
    assignment that rebinds the handle to the new round id, branch/worktree,
    selected lineage, and artifact paths. The new assignment must tell the
    subagent to reload current state and treat prior round details as historical
    context only.
- Same-round retry should first go back to the prior compatible role subagent:
  planner retries to the planner, whole-round implementation retries to the
  implementer, integration retries to the integration implementer, worker
  retries to the named worker, and review-only retries to the reviewer.
- A planner handle that produced `roadmap-update-request.md` may be reused for
  the resulting `update-roadmap` stage after rebinding it to the
  roadmap-update branch/worktree because this remains the same runtime role.
- New rounds should first use an idle cross-round compatible same-role subagent
  when the host exposes one. Round merge makes a finished handle idle for
  same-role reuse; it is not by itself a reason to close the handle.
- Resume a closed but resumable compatible subagent when the host supports it;
  otherwise send the next task to an idle compatible subagent. If no compatible
  prior subagent exists, if the prior subagent is non-observable, stale,
  role-mismatched, or unavailable, use a fresh subagent.
- Do not reuse a subagent across role boundaries. In particular, a reviewer may
  not implement fixes from its own review, an implementer may not review its
  own work, and a recovery investigator may not become the substantive stage
  author or reviewer.
- The `recovery-investigator` runtime role is
  `orchestrator/roles/recovery-investigator.md`.
- The `recovery-investigator` may not act as the substantive stage reviewer.
- Never interrupt a live subagent.
- Never set a cancellation timeout on a live subagent. Non-interrupting
  observation intervals are allowed for liveness checks.
- Wait for the subagent to finish before continuing.
- Close or release host subagent handles only after they are finished and no
  longer compatible with same-role reuse, such as when the control plane is
  terminal with no expected same-role work, the role prompt or control-plane
  contract changes incompatibly, a role boundary would be crossed, recovery
  marks the handle stale or unusable, or the host requires release for resource
  management.
- Do not close a healthy finished handle solely because a round merged or a
  roadmap update cleared. Keep it available for later same-role work unless one
  of the close conditions above applies.
- Do not close a live compatible subagent to force progress; first consume its
  controller-visible artifacts or enter the recovery ladder below.
- After 50 tool calls in one delegated stage, pause for assessment before
  continuing or re-dispatching.
- If a live subagent becomes non-observable, do not hang forever and do not
  interrupt it. Re-read controller-visible artifacts, check any host-provided
  non-interrupting status surface, and enter the recovery ladder when three
  consecutive observations show no status change, no artifact progress, and no
  trustworthy liveness signal.
- Do not convert a failed role-stage launch directly into terminal blockage
  unless `recovery-investigator` has been attempted or deterministically ruled
  out, and no same-round recovery, re-observation, or different-mechanism
  re-dispatch remains lawful.
- If a required `orchestrator/roles/<role>.md` file is missing, stop and record
  the exact controller error in
  `orchestrator/state.json.resume_errors.controller` instead of inventing one.

## The Orchestrator May Not Do

- author role-owned artifacts defined by `orchestrator/artifact-manifest.md`
  and the repo-local schemas
- make changes classified as semantic roadmap updates under
  `orchestrator/active-roadmap-bundle.md` without delegated `update-roadmap`
  and reviewer approval
- choose worker ownership boundaries on its own
- perform substantive code integration or refresh itself
- approve work without a round-level reviewer

## If Real Subagents Are Unavailable

Record the precise blockage in
`orchestrator/state.json.resume_errors.controller`, then tell the user the
runtime skill cannot honor its delegation contract in the current environment.
