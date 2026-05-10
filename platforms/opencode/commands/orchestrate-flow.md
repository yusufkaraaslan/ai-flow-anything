---
name: orchestrate-flow
description: Orchestrate parallel implementation of ALL task flows for a task using git worktrees and subagents. Run after design-flow. Use when the task has 3+ task flows with independent subsets.
argument-hint: <task-name> [--no-commit]
---

# /orchestrate-flow

This command runs the **parallel orchestration flow** for a task. It:
1. Reads all task flow files and builds a dependency graph
2. Presents a wave-based execution plan for your approval
3. Launches subagents (implement-flow in auto-proceed mode) in parallel within each wave
4. Cherry-picks worktree commits, resolves conflicts, validates
5. Presents a consolidated report as the final gate

## Usage

```
/orchestrate-flow user-authentication
/orchestrate-flow shield-power-up --no-commit
```

## Prerequisites

- Task must have a signed-off task-design.md (run `/design-flow` first)
- Task flow files must exist with dependency graph
- Git worktree support must be available

## Phases

| Phase | Action | Gate |
|-------|--------|------|
| 1. PLAN | Group task flows into dependency waves, present execution plan | [A]/[F]/[R] STANDARD |
| 2. EXECUTE | Create worktrees, launch subagents in parallel per wave | Auto (no gate) |
| 3. MERGE | Cherry-pick commits, resolve conflicts, validate, generate report | [A]/[F]/[R] CRITICAL |
| 4. COMMIT | Stage and commit; cleanup worktrees | Auto (no gate) |

## Example

```
/orchestrate-flow user-authentication

→ Phase 1: GROUP & PLAN
  Wave 1: user-session-store, auth-interfaces       [2 parallel]
  Wave 2: auth-service, login-form                   [2 parallel, depends on Wave 1]
  Wave 3: api-integration                            [depends on Wave 2]
  Wave 4: tests                                      [depends on all]

  [A]ccept / [F]eedback / [R]eject?
  > A

→ Phase 2: EXECUTE
  Wave 1: ✓ user-session-store (tests 8/8)  ✓ auth-interfaces (tests 3/3)
  Wave 2: ✓ auth-service (tests 12/12)      ✓ login-form (tests 5/5)
  Wave 3: ✓ api-integration (tests 6/6)
  Wave 4: ✓ tests (tests 15/15)

→ Phase 3: MERGE
  Consolidated report: 5/5 task flows, 44/44 tests passing, 0 conflicts
  [A]ccept / [F]eedback / [R]eject?
```

For the full flow, see `.ai-workflow/flows/orchestrate-flow.md`.
