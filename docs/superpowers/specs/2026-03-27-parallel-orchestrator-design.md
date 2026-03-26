# Parallel Orchestrator Skill Design

## Goal

Extend the orchestrator skill set so it can run:

- multiple independent roadmap items in parallel as separate rounds
- multiple explicit worker slices in parallel inside a single round stage

The design must keep concurrency explicit in repo-local state and artifacts
rather than inferred from chat context or controller heuristics.

## Current Problems

- The runtime contract is single-round and single-active-subagent.
- `orchestrator/state.json` models only one active round at a time.
- The roadmap format cannot explicitly declare safe parallel work.
- The planner role is written as a strictly sequential round planner.
- The implementer role owns one undivided round, with no worker-slice contract.
- The reviewer and merger contracts assume one linear round lifecycle.
- The scaffold skill generates a serial-only contract for new repositories.

These constraints make the current skill set predictable, but they prevent safe
parallel execution even when independent work is obvious.

## Decision

Parallelism will be opt-in and controller-visible.

The controller must never infer that two roadmap items or two worker slices are
safe to run concurrently. Parallel execution is allowed only when repo-local
artifacts explicitly authorize it.

The base rules are:

- roadmap items remain serial unless the roadmap marks them parallel-safe
- multiple rounds may run concurrently only when dependency and grouping rules
  allow it
- worker fan-out inside a round is allowed only when the round plan explicitly
  defines safe slices, ownership, and recomposition
- review and merge remain conservative and repo-visible

## Compatibility Rule

The runtime skill should remain usable on already-scaffolded serial
repositories.

Compatibility requirements:

- missing parallel metadata in `roadmap.md` means serial execution
- missing additive parallel fields in `orchestrator/state.json` should be
  initialized by the controller to safe serial defaults instead of treated as
  fatal corruption
- existing repositories with one active round should continue to run unchanged
  unless repo-local artifacts are revised to opt into concurrency

The scaffold skill will generate the full new schema for freshly scaffolded
repositories.

## Roadmap Contract

The active roadmap bundle remains the scheduler's source of truth.

Each roadmap item keeps:

- ordered item number
- status marker
- `Depends on:`
- `Completion notes:`

Each item also gains explicit concurrency metadata:

- `Parallel safe:` `yes` or `no`
- `Parallel group:` stable identifier or `none`
- `Merge order:` `after-deps` or `fixed-order`

Default behavior:

- unmarked items are treated as `Parallel safe: no`
- items with unfinished dependencies cannot start
- items in different parallel groups do not run together unless the roadmap
  explicitly allows that grouping pattern

This keeps concurrency decisions in the human-reviewed roadmap file instead of
controller inference.

## State Model

`orchestrator/state.json` must grow from a single-active-round model into a
bounded multi-round scheduler state.

The new machine-oriented state should add:

- `max_parallel_rounds`: bounded concurrency cap
- `active_rounds`: array of round records currently in flight
- `pending_merge_rounds`: approved rounds waiting for merge order or base-branch
  readiness
- `resume_errors`: controller-level or per-round recoverable errors

Each `active_rounds[]` record should include:

- `round_id`
- `roadmap_item_id` or stable task identifier
- `current_task`
- `stage`
- `branch`
- `worktree_path`
- `active_round_dir`
- `round_artifacts`
- `depends_on_rounds`
- `parallel_group`
- `merge_ready`
- `resume_error`

The controller may still expose `active_round_id` as a convenience pointer for
resume UX, but machine state must treat `active_rounds[]` as the canonical
source for live concurrency.

## Scheduling Model

The runtime scheduler remains deterministic.

Scheduling rules:

- launch parallel rounds only for roadmap items explicitly marked parallel-safe
- require all declared `Depends on:` entries to be `done` before scheduling an
  item
- enforce `max_parallel_rounds` as a hard cap
- keep each round on its own branch, worktree, and artifact directory
- allow different rounds to occupy different stages at the same time
- never skip the per-round stage order

Per-round stage order remains:

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `merge`
6. `update-roadmap`
7. `done`

The runtime may advance multiple rounds concurrently, but it may not invent
parallel stages inside a round unless the plan explicitly authorizes worker
fan-out.

## Parallel Round Model

Parallel branches are modeled as multiple live rounds.

For each live round, the controller must preserve:

- one round id
- one branch
- one worktree
- one round directory
- one canonical round `plan.md`
- one canonical round `review.md`
- one canonical round `merge.md`

Reviewer approval does not imply immediate merge. A round may become
`approved-pending-merge` when:

- another approved round must merge first because of explicit ordering
- the branch must be rebased or refreshed against the updated base branch
- a dependency round is not yet merged

The controller merges only rounds that are both approved and merge-ready under
the declared dependency graph.

## Parallel Stage Worker Model

Stage-worker fan-out is opt-in inside a round.

The planner may define a worker plan only when the round can be decomposed into
disjoint owned slices.

The plan must declare:

- worker identifiers
- each worker's exact owned files or module boundary
- required artifacts
- integration order
- any slice-specific checks
- the rule for recomposing worker outputs into the round branch

When `plan.md` declares worker slices, the controller may launch multiple
implementer workers in parallel for that round.

The controller must not invent worker slices on its own.

If the plan does not define safe slices, the round stays single-worker.

## Round Artifacts

The existing round-level artifacts remain canonical:

- `selection.md`
- `plan.md`
- `implementation-notes.md`
- `review.md`
- `review-record.json`
- `merge.md`

Parallel worker execution adds optional worker-scoped artifacts:

- `worker-plan.json`
- `workers/<worker-id>/assignment.md`
- `workers/<worker-id>/implementation-notes.md`
- `workers/<worker-id>/review.md`

The integrated round still ends with one canonical round-level review record
and one merge note. Worker artifacts are supporting evidence, not replacements
for the round contract.

## Role Contract Changes

### `guider`

The guider may nominate more than one next roadmap item only when the roadmap
explicitly marks them as parallel-safe and dependency-clean.

### `planner`

The planner remains responsible for the round plan, but may now author:

- a sequential single-worker plan
- a parallel worker plan with explicit ownership boundaries

The planner must not mark ambiguous work as parallel merely because it looks
independent.

### `implementer`

The implementer role must support two modes:

- whole-round implementation
- worker-slice implementation inside a planner-authored worker plan

Worker implementers own only their assigned slice and must not rewrite the
overall plan or claim round approval.

### `reviewer`

The reviewer must approve or reject the integrated round outcome, not just the
worker slices in isolation.

Worker review artifacts may support the decision, but the canonical approval
still happens at the round level.

### `merger`

The merger prepares only approved, merge-ready rounds. It must respect the
declared merge ordering and base-branch freshness requirements for parallel
rounds.

## Recovery and Retry

Parallelism must not weaken recovery rules.

`retry-subloop.md` should explicitly define:

- whether retry is allowed per round, per worker slice, or both
- whether a failed worker review requires full-round replanning or only
  slice-level retry
- how a reviewed-but-not-merged round re-enters execution after base-branch
  drift

The controller should continue to treat incidental delegation failure as a
recovery problem, not an excuse to silently drop a round or worker slice.

## Skill Changes

### `scaffold-orchestrator-loop`

The scaffold skill will:

- scaffold the expanded `state.json` schema
- scaffold roadmap templates with parallel metadata fields
- scaffold role templates that describe round concurrency and worker-slice
  ownership
- scaffold retry-contract guidance covering both round retry and worker retry

### `run-orchestrator-loop`

The runtime skill will:

- normalize missing additive parallel fields to safe serial defaults
- load concurrency rules from the active roadmap bundle and round artifacts
- manage multiple live rounds concurrently within the configured cap
- launch parallel implementer workers only from explicit planner-authored worker
  plans
- keep review and merge decisions round-level and explicit

## README Changes

The README should describe the new model precisely:

- rounds may run in parallel only when the roadmap marks them parallel-safe
- stage-worker fan-out is opt-in and defined by round plans
- each round still uses one branch, one worktree, and one canonical review and
  merge record
- unmarked repositories continue to behave serially

## Verification

Verification should focus on contract integrity rather than executable code.

Required checks should include:

- `git diff --check`
- `rg` checks proving the serial-only language has been replaced where needed
- `rg` checks proving the new roadmap metadata and state keys are documented
- checks that scaffold assets, live skill docs, and README agree on the same
  concurrency model
- discovery verification that both skills still list correctly through the
  `skills` CLI

## Risks

- Parallel metadata that is underspecified will create ambiguous scheduler
  behavior.
- Worker-slice ownership that overlaps in practice will create merge churn and
  rework.
- A state schema that tries to preserve too much single-round compatibility can
  become confusing or internally inconsistent.
- If README, scaffold assets, and runtime skill drift, users will not know what
  the supported concurrency model actually is.

## Non-Goals

- implicit controller inference of safe parallel work
- unbounded fan-out across every pending roadmap item
- replacing round-level review with worker-level approvals
- hiding concurrency state in session memory instead of repo-local artifacts
