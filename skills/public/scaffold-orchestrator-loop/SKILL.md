---
name: scaffold-orchestrator-loop
description: Use when a repository needs a new repo-local orchestrator workflow scaffolded from a goal, including an initial roadmap, `orchestrator/` control files, a verification contract, and repo-local `.codex/agents` role definitions before delegated runtime execution begins.
---

# Scaffold Orchestrator Loop

## Overview

Create the repository-local control plane for the orchestrator workflow. Review the goal and repository first, scaffold a tailored `orchestrator/` directory plus repo-local `.codex/agents/` role files, prepare the repo for per-round worktrees, and stop after the initial checkpoint commit.

## Workflow

1. Survey the repository and the goal.
2. Build the initial roadmap.
3. Scaffold and tailor the `orchestrator/` contract.
4. Commit the initial checkpoint and stop.

## Step 1: Survey the Repository

Inspect the repository before writing anything:

- top-level files and docs
- existing tests and verification commands
- current Git status
- whether Git is initialized already

If Git is missing, initialize it with `git init -b main`.

## Step 2: Build the Initial Roadmap

Read [roadmap-generation.md](references/roadmap-generation.md), then replace the roadmap template with a repo-specific ordered roadmap. Keep later items coarse and make the next item concrete.

## Step 3: Scaffold the Repo Contract

Read [repo-contract.md](references/repo-contract.md) and [verification-contract.md](references/verification-contract.md).

Then:

- copy [assets/orchestrator](assets/orchestrator) into the target repo root as `orchestrator/`
- copy [assets/.codex/agents](assets/.codex/agents) into the target repo root as `.codex/agents/`
- replace template placeholders with repo-specific content
- tune the repo-local role agents if the goal or repo needs stronger guidance
- ensure `.codex/agents/` is tracked in git; if the repo ignores `.codex/`, add an exception for the scaffolded role agents
- ensure `.worktrees/` is ignored so future rounds can use dedicated worktrees

Keep `state.json` machine-oriented. Put reasoning and reviewable content in the human-facing files under `orchestrator/`.

## Step 4: Create the Checkpoint Commit

After the scaffolded files are tailored:

- stage the new orchestrator files and the scaffolded `.codex/agents/` role agents
- create the initial checkpoint commit
- stop after setup

Do not start implementation rounds. Runtime orchestration belongs to `$run-orchestrator-loop`, not this skill.

## Resources

- [repo-contract.md](references/repo-contract.md): required file layout, state schema, and ownership rules
- [roadmap-generation.md](references/roadmap-generation.md): how to derive the initial roadmap from the goal and repo
- [verification-contract.md](references/verification-contract.md): how to tailor `orchestrator/verification.md`
- [assets/orchestrator](assets/orchestrator): templates for the scaffolded controller state
- [assets/.codex/agents](assets/.codex/agents): templates for the scaffolded repo-local role agents
