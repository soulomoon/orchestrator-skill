# Contributing

## Repository Structure

This repository ships two agent-neutral skills under `skills/`:

- `scaffold-orchestrator-loop` — initializes the `orchestrator/` control plane
- `run-orchestrator-loop` — resumes and coordinates the delegated round loop

Supporting files:
- `skills/*/references/` — supporting documentation loaded by the skills
- `skills/scaffold-orchestrator-loop/assets/` — templates copied into target repos
- `docs/superpowers/` — original design specs and implementation plans

## Local Development

Symlink the skill directories into your agent's skill location for live
development:

```bash
ln -s /path/to/orchestratorpattern/skills/scaffold-orchestrator-loop ~/.codex/skills/scaffold-orchestrator-loop
ln -s /path/to/orchestratorpattern/skills/run-orchestrator-loop ~/.codex/skills/run-orchestrator-loop
```

If either path already exists, move or remove the existing entry first.

## Validating Changes

After editing skill files, run these checks:

### Contract Integrity

```bash
# No legacy Codex-specific references in active skill surfaces
! rg -n "\.codex/agents|\.worktrees|codex/round-" \
  skills/run-orchestrator-loop \
  skills/scaffold-orchestrator-loop

# Canonical paths are present
rg -n "orchestrator/roles/|orchestrator/worktrees/|orchestrator/round-" \
  skills/run-orchestrator-loop/SKILL.md \
  skills/scaffold-orchestrator-loop/SKILL.md
```

### Skill Discovery

```bash
npx skills add . --list
# Expected: Found 2 skills
```

### Whitespace and Formatting

```bash
git diff --check
```

## Design Specs

The `docs/superpowers/` directory contains the original design documents:

- `specs/2026-03-26-platform-neutral-orchestrator-design.md` — migration from
  Codex-specific to agent-neutral contract
- `specs/2026-03-27-parallel-orchestrator-design.md` — parallel round and
  worker fan-out design
- `plans/` — step-by-step implementation plans used to build the current skills

These are historical records of design decisions. When making changes, check
that your edits remain consistent with the design rationale.
