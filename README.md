# Orchestrator Skill Set

This repository contains a pair of Codex skills for running a strict repo-local orchestration workflow:

- `scaffold-orchestrator-loop` initializes an `orchestrator/` control plane in a target repository.
- `run-orchestrator-loop` resumes and coordinates the delegated round loop without doing the substantive work itself.

The design is intentionally explicit. State lives in the repository, delegated role prompts are inspectable, and each round uses a dedicated branch and worktree so progress can be resumed without relying on chat history.

## Included Skills

### `scaffold-orchestrator-loop`

Use this skill when a repository does not yet have an `orchestrator/` directory.

It is responsible for:

- surveying the repository and current goal
- generating the first roadmap
- scaffolding `orchestrator/roadmap.md`, `orchestrator/state.json`, `orchestrator/verification.md`, and role prompts
- preparing the repository for per-round worktrees
- creating the initial checkpoint commit

### `run-orchestrator-loop`

Use this skill after setup, when the repository already has an initialized `orchestrator/` directory.

It is responsible for:

- loading persisted orchestration state
- resuming the current round or starting the next one
- delegating `select-task`, `plan`, `implement`, `review`, and `merge` stages to fresh subagents
- updating only controller-owned state
- squash-merging approved rounds and advancing the roadmap

## Workflow Model

The runtime loop is strict about ownership:

- The orchestrator is a controller, not an implementer.
- Each round is linear and uses one branch plus one worktree.
- Review rejection sends the round back to `plan`, then `implement`, then `review` again.
- Resume behavior comes from files in the repository, not hidden session context.

The stage order is:

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `merge`
6. `update-roadmap`
7. `done`

## Repo-Local Contract

The scaffolded repository gets a visible top-level `orchestrator/` directory with these core files:

```text
orchestrator/
├── roadmap.md
├── state.json
├── verification.md
├── roles/
│   ├── guider.md
│   ├── planner.md
│   ├── implementer.md
│   ├── reviewer.md
│   └── merger.md
└── rounds/
```

Key ideas behind that contract:

- `state.json` stays machine-oriented and tracks the active round, stage, branch, worktree, and resume errors.
- Human-facing reasoning stays in `roadmap.md`, role prompts, and round artifacts.
- Each round folder stores delegated artifacts such as `selection.md`, `plan.md`, `implementation-notes.md`, `review.md`, and `merge.md`.

## Repository Layout

This repository ships the skills under `skills/public`:

```text
skills/public/
├── scaffold-orchestrator-loop/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   ├── assets/orchestrator/
│   └── references/
└── run-orchestrator-loop/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
```

- `SKILL.md` is the entrypoint instruction file.
- `references/` contains supporting rules for roadmap generation, state transitions, resume behavior, and merge boundaries.
- `assets/` contains the starter `orchestrator/` contract used by the scaffold skill.

## Typical Usage

1. Add or install these skills in your Codex environment.
2. In a target repository, invoke `scaffold-orchestrator-loop` with the high-level goal.
3. Review the generated `orchestrator/` contract and initial checkpoint commit.
4. Invoke `run-orchestrator-loop` to start or resume the delegated round loop.
5. Let the runtime skill continue until roadmap items are complete or a recorded controller error blocks progress.

## Development Notes

- Round branches are expected to use the `codex/` prefix.
- Round worktrees live under `.worktrees/` in the target repository.
- Reviewer approval is required before merge.
- Merge strategy is squash merge into the recorded base branch.

## Source Documents

The design and implementation plan used to build this repo are checked in under:

- `docs/superpowers/specs/2026-03-13-orchestrator-loop-design.md`
- `docs/superpowers/plans/2026-03-13-orchestrator-skill-set.md`
