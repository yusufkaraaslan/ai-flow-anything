---
description: Orchestrate parallel implementation of ALL task flows for a task using git worktrees and parallel Claude Code subagents. Run after design-flow. Use when the task has 3+ task flows with independent subsets.
argument-hint: <task-name> [--no-commit]
---

# /orchestrate-flow

Runs the **parallel orchestration flow** for a task. Loads `.ai-workflow/flows/orchestrate-flow.md` and executes it.

## Usage

```
/orchestrate-flow user-authentication
/orchestrate-flow shield-power-up --no-commit
```

## Prerequisites

- Task must have a signed-off `flow-storage/tasks/{task-name}/design/task-design.md` (run `/design-flow` first)
- Task flow files must exist in `flow-storage/tasks/{task-name}/implement/flow-plan/` with `depends-on` frontmatter
- Git worktree support must be available
- `.claude/agents/orchestrate-implementer.md` must be installed (auto-installed by `/ai-flow-anything init`; verify the Agent tool lists `orchestrate-implementer` as a `subagent_type`)
- Working directory must be clean (`git status` reports no changes)

## Phases

| Phase | Action | Gate |
|-------|--------|------|
| 1. PLAN | Group task flows into dependency waves, present execution plan | [A]/[F]/[R] STANDARD |
| 2. EXECUTE | Launch `orchestrate-implementer` subagents in parallel per wave (auto-worktree via `isolation: worktree`) | Auto (no gate) |
| 3. MERGE | Cherry-pick subagent commits, resolve conflicts, validate, generate report | [A]/[F]/[R] CRITICAL |
| 4. COMMIT | Stage and commit; sub-agent branches stay until manual prune | Auto (no gate) |

## How parallelism works on Claude Code

Phase 2.2 issues N Agent-tool calls in a single message, all with `subagent_type: orchestrate-implementer`. Claude Code dispatches them concurrently, each in its own auto-created worktree under `.claude/worktrees/orchestrate-implementer-<id>/`. The parent receives all reports when the last subagent in the wave finishes, then proceeds to the next wave.

## Example

```
/orchestrate-flow user-authentication

→ Phase 1: PLAN
  Wave 1: user-session-store, auth-interfaces       [2 parallel]
  Wave 2: auth-service, login-form                  [2 parallel, depends on Wave 1]
  Wave 3: api-integration                           [depends on Wave 2]
  Wave 4: tests                                     [depends on all]

  [A]ccept / [F]eedback / [R]eject?
  > A

→ Phase 2: EXECUTE
  Wave 1: ✓ user-session-store (commit a1b2c3d, tests 8/8)
          ✓ auth-interfaces    (commit e4f5g6h, tests 3/3)
  Wave 2: ✓ auth-service       (commit i7j8k9l, tests 12/12)
          ✓ login-form         (commit m0n1o2p, tests 5/5)
  Wave 3: ✓ api-integration    (commit q3r4s5t, tests 6/6)
  Wave 4: ✓ tests              (commit u6v7w8x, tests 15/15)

→ Phase 3: MERGE
  Consolidated report: 6/6 task flows, 49/49 tests passing, 0 conflicts
  [A]ccept / [F]eedback / [R]eject?
```

For the full flow, see `.ai-workflow/flows/orchestrate-flow.md`. For Claude-specific phase notes (the auto-worktree behaviour, the cherry-pick branch lookup), see `.ai-workflow/platforms/claude/flow-skills/orchestrate-flow/SKILL.md` or its installed copy at `.claude/skills/orchestrate-flow/SKILL.md`.
