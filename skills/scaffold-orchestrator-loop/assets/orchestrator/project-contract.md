# Project Contract

This file records repo-wide invariants shared by every roadmap family and
round. Keep roadmap revisions focused on current coordination; point here for
stable contracts instead of restating them in every role or roadmap file.

These invariants describe the scaffolded orchestrator control plane itself.
During scaffold alignment, append target-repository domain invariants here only
when they apply across roadmap families. Keep roadmap-specific overrides in the
active roadmap bundle.

## Stable Interfaces

- Repo-local control plane: `orchestrator/` is the durable orchestration Module.
  Its shared Interface is the tracked file set named by
  `orchestrator/artifact-manifest.md`, including `state.json`,
  `state-schema.md`, `artifact-manifest.md`, `project-contract.md`,
  `active-roadmap-bundle.md`, `role-contract.md`, the schema files, role files,
  round artifacts, roadmap-update artifacts, roadmap families, and worktree
  staging.
- Controller state: `state.json` is machine-oriented and minimal. It records
  `contract_version: "orchestrator-v2"`, active roadmap metadata,
  `controller_stage`, `max_parallel_rounds`, `active_rounds`,
  `roadmap_update`, `resume_errors`, and `retry`. Do not persist derived
  mirror fields such as preferred round, merge readiness, summary state,
  worker mode, artifact paths, roadmap style, or top-level resume error.
- Active roadmap bundle: `state.json.roadmap_dir` names the only live roadmap
  revision directory. That directory contains `roadmap.md`,
  `roadmap-view.json`, and `verification.md`; the family directory contains
  `roadmap-history.md`. Top-level roadmap, verification, and retry-policy
  pointer stubs are not supported.
- Strategy-backlog roadmap adapter: strategy-backlog is the only supported
  roadmap shape. `roadmap-view.json` with `schema_version:
  "roadmap-view-v1"` is the machine-readable view for milestone ids, direction
  ids, dependencies, terminal status, and closeout anchors. `roadmap.md` stays
  the human coordination narrative.
- Artifact and path resolution: `artifact-manifest.md` owns shared file layout,
  round artifact keys, worker artifact paths, roadmap-update artifact paths,
  and live-vs-archived path resolution.
- Role Interface: `role-contract.md` owns shared role inputs, ownership rules,
  output rules, boundaries, and self-checks. Files under `roles/` should carry
  only role-specific behavior.
- Round records: `selection-record.json` owns selected round lineage and
  scheduler fields. `round-plan-record.json` owns planner-authored machine
  planning data and optional worker fan-out. `review-record.json` and
  `closeout-record.json` follow `round-finalization-schema.md`.
- Semantic roadmap updates: `roadmap-update-schema.md` owns
  `state.json.roadmap_update`, merged-round and planner-request triggers,
  update branch/worktree conventions, update and review artifacts, rejection
  handling, and activation.
- Retry policy: shared retry mechanics live in runtime references.
  Roadmap-specific retry overrides live under active-bundle `verification.md`
  `## Roadmap Overrides`; do not create a separate required retry-policy file.

## Alignment Invariants

- The orchestrator is a controller, not an implementer. It owns controller
  state, branch/worktree coordination, round finalization, and activation of
  approved semantic roadmap updates.
- The basic serial workflow is the default runtime front door:
  `plan` -> `implement` -> `review` -> `finalize-round`, followed by terminal
  roadmap recheck. Advanced recovery, worker fan-out, parallel execution, and
  semantic roadmap-update machinery load only when their triggers are present.
- The planner owns roadmap stewardship: normal task selection, round planning,
  and semantic `update-roadmap` authoring. It writes `selection-record.json`,
  `plan.md`, and `round-plan-record.json` for selected implementable rounds, or
  `roadmap-update-request.md` when current evidence shows the active roadmap
  must first be split or resequenced. During `update-roadmap`, the same role
  writes `roadmap-update.md` and the proposed roadmap revision.
- Reviewer approval gates merge. Rejected reviews record a machine
  `retry_target` of `implement`, `plan`, or `blocked` plus specific required
  changes.
- `finalize-round` is controller-owned. It applies reviewer-approved
  status-only closeout when needed, writes `closeout-record.json`, derives
  merge admissibility from machine records and base freshness, performs squash
  merge bookkeeping, removes the live round from `active_rounds`, and dispatches
  semantic `update-roadmap` only when required.
- Status-only closeout may change only approved milestone status markers,
  compact completion pointers, and compact roadmap-history entries through
  `roadmap-view.json` anchors. It must not change future coordination,
  milestone or direction meaning, sequencing, parallel lanes, extraction scope,
  verification meaning, or retry policy.
- Semantic roadmap updates are delegated, reviewable, and serialized through
  `state.json.roadmap_update`. They may come from a merged round or from a
  planner-requested pre-implementation split, but only one semantic roadmap
  update may be active. Approved updates publish a new roadmap revision before
  activation.
- Used roadmap revisions are durable history. Do not rewrite a used revision
  except for reviewer-approved status-only closeout in the canonical round
  worktree before merge.
- Legacy compatibility layers are retired. Do not reintroduce legacy mirror
  fields, legacy roadmap style, `roadmap_style`, compact completion summary
  state, persisted merge readiness, prose selection handoffs, top-level pointer
  stubs, or a dedicated merge-preparation role.
- Worktrees live under `orchestrator/worktrees/` and are ignored by tracked
  ignore rules. Durable machine state, role files, roadmap bundles, and round
  artifacts stay in the main repo-visible control plane.

## Verification Anchors

- Verify the scaffolded `orchestrator/` file set and path-resolution rules
  against `artifact-manifest.md`.
- Verify `state.json` against `state-schema.md`: minimal top-level fields,
  `contract_version: "orchestrator-v2"`, no retired compatibility fields, live
  rounds only in `active_rounds[]`, controller blockage only in
  `resume_errors.controller`, and no persisted merge-readiness or worker-mode
  mirrors.
- Verify active roadmap bundle resolution through `state.json.roadmap_dir`.
  Treat missing bundle files, invalid `roadmap-view.json`, duplicate ids,
  unknown statuses, missing anchors, or `roadmap.md` / `roadmap-view.json`
  conflicts as controller errors, not terminal roadmaps.
- Verify terminal completion only when every `roadmap-view.json` milestone is
  `done`, `active_rounds` is empty, `roadmap_update` is `null`, and no unresolved
  resume errors remain.
- Verify every machine record against its schema before relying on it:
  `selection-record.json`, `round-plan-record.json`, `review-record.json`,
  `closeout-record.json`, and roadmap-update records.
- Verify selected lineage matches across selection, plan, review, and closeout
  records. Treat duplicated lineage outside the schema-governed records only as
  integrity checks, not authority.
- Verify status-only closeout selectors resolve through `roadmap-view.json`
  anchors and that every applied edit appears in the approved
  `review-record.json`. After base refresh, revalidate `closeout-record.json`
  before merge.
- Verify semantic roadmap updates use `roadmap-update-schema.md`, record the
  correct trigger, have reviewer approval before activation, and clear
  `state.json.roadmap_update` only after the approved revision is active.
- Verify new roadmap families preserve prior families, prior revisions, prior
  rounds, worktrees, and role files unless an approved migration explicitly says
  otherwise.
- Baseline command discovery is target-repository specific. The active
  revision's `verification.md` must record discovered tests, lint/format,
  build/typecheck, documentation/example checks, or explicit manual checks when
  automation is absent. Do not duplicate universal role, schema, closeout, or
  roadmap-update rules there.

## Update Rule

Update this file only when the repo-wide invariant itself changes. When a
roadmap temporarily narrows or extends an invariant, record the override in the
active roadmap bundle and keep the durable rule here.
