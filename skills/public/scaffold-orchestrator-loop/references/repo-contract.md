# Repo Contract

Scaffold a visible top-level `orchestrator/` directory plus repo-local `.codex/agents/` role definitions in the target repository.

## Required Files

- `orchestrator/roadmap.md`
- `orchestrator/state.json`
- `orchestrator/verification.md`
- `.codex/agents/orchestrator-guider.toml`
- `.codex/agents/orchestrator-planner.toml`
- `.codex/agents/orchestrator-implementer.toml`
- `.codex/agents/orchestrator-reviewer.toml`
- `.codex/agents/orchestrator-merger.toml`
- `orchestrator/rounds/`

## `state.json` Schema

Use a small machine-oriented state file.

| Key | Type | Meaning |
| --- | --- | --- |
| `base_branch` | string | Branch that accepted rounds merge into |
| `active_round_id` | string or null | Current round identifier |
| `stage` | string | `done`, `select-task`, `plan`, `implement`, `review`, `merge`, or `update-roadmap` |
| `current_task` | string or null | Selected roadmap item summary |
| `branch` | string or null | Active round branch name |
| `worktree_path` | string or null | Active round worktree path |
| `active_round_dir` | string or null | Folder under `orchestrator/rounds/` for the active round |
| `round_artifacts` | object | Artifact path map for the active round |
| `last_completed_round` | string or null | Most recently accepted round |
| `resume_error` | string or null | Last recoverable controller error |

## File Ownership

- The orchestrator may update `orchestrator/state.json` and round bookkeeping only.
- Delegated role agents author roadmap content, round artifacts, review records, and merge notes.
- The guider owns task selection and roadmap updates.

## Compatibility and Migration

- New repos should scaffold the namespaced `.codex/agents/orchestrator-<role>.toml` files as the default contract.
- Runtime remains compatible with legacy `orchestrator/roles/<role>.md` files on a per-role basis.
- During migration, keep each legacy role file until the matching `.codex/agents/orchestrator-<role>.toml` file exists.

## Round Artifacts

Each round folder should contain delegated artifacts only. Start with these names:

- `selection.md`
- `plan.md`
- `implementation-notes.md`
- `review.md`
- `merge.md`

Add more files only when a round needs them.

## Worktree Preparation

Prepare the repository for round worktrees during setup:

- Ensure `.worktrees/` is gitignored.
- Ensure the scaffolded `.codex/agents/` files stay tracked; if `.codex/` is ignored, carve out an exception for the orchestrator role agents.
- Do not place machine state inside `.worktrees/`.
- Keep `orchestrator/` in the main checkout so resume state stays visible.
