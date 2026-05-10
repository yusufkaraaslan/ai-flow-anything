---
name: orchestrate-flow
description: Use when the developer says "orchestrate X", "implement all task flows for X", "run all task flows in parallel for X", or "build everything for X". Orchestrates the parallel implementation of ALL task flows for a task using isolated git worktrees and subagents, in 4 phases with 2 review gates [A]/[F]/[R] (one after the execution plan, one after merged implementation). Requires a signed-off task-design.md with task flow dependency graph. The full flow is in .ai-workflow/flows/orchestrate-flow.md — load that file and execute it.
license: MIT
metadata:
  audience: developers
  workflow: task-development
  phase: orchestrate
---

# orchestrate-flow (OpenCode wrapper)

This skill runs the **parallel orchestration** of all task flows for a task. The full flow lives in `.ai-workflow/flows/orchestrate-flow.md`. Read it and execute.

## When to invoke

- "Orchestrate {task-name}"
- "Implement all task flows for {task-name}"
- "Run all task flows in parallel for {task-name}"
- "Build everything for {task-name}"

The developer may also say "implement {task-name}" (without specifying a task flow). If the task has 3+ task flows with independent subsets, suggest orchestrate-flow as the faster option.

## Hard preconditions

1. **task-design.md signed off** at `flow-storage/tasks/{task-name}/design/task-design.md`
2. **Task flow files exist** in `flow-storage/tasks/{task-name}/implement/flow-plan/` with `depends-on` frontmatter
3. **Git worktree support** (`git worktree --help` works on this machine)
4. **Working directory clean** (`git status` reports no changes). Phase 3 MERGE cherry-picks into the main WD; a dirty WD is unsafe. If dirty, offer Stash / Commit-first / Abort. See `.ai-workflow/flows/orchestrate-flow.md` Prerequisites for the script.

If any precondition fails, STOP and direct developer to design-flow, implement-flow (sequential), or `git stash` as appropriate.

## What this flow does

**4 phases, 2 gates.** Phases 2 and 4 auto-execute after the preceding gate is accepted.

### Phase 1: PLAN *(GATE TYPE B — STANDARD)*

- **1.1 LOAD** — Read all task flow files, design docs, KB context.
- **1.2 GROUP** — Topological sort by `depends-on` → compute dependency waves. Validate no two task flows in the same wave share files.
- **1.3 SCHEDULE** — Generate execution plan: waves, task flow groupings, worktree paths.
→ **Gate 1: Execution plan review** — developer adjusts grouping/reordering.

### Phase 2: EXECUTE *(auto — no gate)*

For each wave sequentially:
- **2.1 PREPARE WORKTREES** — `git worktree add` for each task flow in the wave.
- **2.2 LAUNCH SUBAGENTS** — Launch implement-flow in sub-agent mode (no gates) for each task flow in the wave, in parallel. Subagents auto-proceed through READ → PLAN → IMPLEMENT → AUTO-VALIDATE → DOC-SYNC → UPDATE TASK FLOW → COMMIT in their worktree.
- **2.3 MONITOR** — Track progress, handle failures.
- **2.4 WAIT FOR WAVE** — All subagents complete before next wave.

No blocking gates in this phase — the plan was already approved.

### Phase 3: MERGE *(GATE TYPE C — CRITICAL)*

- **3.1 COLLECT** — Gather reports + commit SHAs from all subagents.
- **3.2 CHERRY-PICK** — Cherry-pick worktree commits into main branch; resolve conflicts.
- **3.3 VALIDATE** — Run full test suite, integration checks.
- **3.4 UPDATE RECORDS** — Transform all task flow files to knowledge records; update TDD, edge-cases, PATTERNS.
- **3.5 GENERATE REPORT** — Consolidated orchestration report.
→ **Gate 2: Final implementation review** — the ONE AND ONLY implementation gate. Developer reviews merged code, consolidated report, validation results.

### Phase 4: COMMIT *(auto — no gate)*

- **4.1 STAGE** — Stage all implementation files + updated knowledge records.
- **4.2 COMMIT** — `feat({task-name}): merged parallel implementation — {N} task flows in {W} waves`. Clean up `git worktree remove`. Opt-out: `--no-commit`.

## Hard rules

- Do not write any task code from this flow. Each task flow is implemented by a subagent running implement-flow.
- Do not auto-accept any phase gate (Rule 6). Only 2 gates exist: Phase 1 (PLAN) and Phase 3 (MERGE).
- Subagents auto-proceed through implement-flow without gates (Rule 16 — sub-agent gate suppression).
- Each subagent works in its own git worktree (Rule 17 — git worktree isolation).
- Read `.ai-workflow/universal/rules.md` and `.ai-workflow/rules.md` before starting.

## Platform status: experimental on OpenCode

OpenCode documents parallel subagent invocation via the Task tool ([opencode.ai/docs/agents](https://opencode.ai/docs/agents/)) and the General subagent supports "running multiple units of work in parallel." However, OpenCode does **not** document a mechanism for per-subagent custom working directories — the assumption that subagents operate inside their own worktree path is unverified.

Before running orchestrate-flow on real work, prototype with two parallel subagents on a 2-task-flow task and confirm they each operate inside distinct `.ai-workflow/worktrees/{task-name}/*` directories. If they step on each other in the main project root, fall back to sequential `implement-flow` until OpenCode documents per-subagent CWD support, OR use the `git -C <worktree-path>` prefix workaround described in `.ai-workflow/flows/orchestrate-flow.md` § Platform Requirements.

For full instructions read `.ai-workflow/flows/orchestrate-flow.md`.
