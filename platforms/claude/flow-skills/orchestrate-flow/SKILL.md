---
name: orchestrate-flow
description: Use when the developer says "orchestrate X", "implement all task flows for X", "run all task flows in parallel for X", or "build everything for X". Orchestrates the parallel implementation of ALL task flows for a task using isolated git worktrees and parallel subagents (Claude Code Agent tool with the `orchestrate-implementer` custom subagent), in 4 phases with 2 review gates [A]/[F]/[R] (one after the execution plan, one after merged implementation). Requires a signed-off task-design.md with task flow dependency graph. The full flow is in .ai-workflow/flows/orchestrate-flow.md — load that file and execute it.
argument-hint: <task-name> [--no-commit]
---

# orchestrate-flow (Claude Code wrapper)

This skill runs the **parallel orchestration** of all task flows for a task. The full flow lives in `.ai-workflow/flows/orchestrate-flow.md`. Read it and execute. This wrapper exists to (a) give the Claude Code agent strong description-driven matching for natural-language orchestrate requests and (b) document the Claude-native subagent invocation pattern.

## When to invoke

- "Orchestrate {task-name}"
- "Implement all task flows for {task-name}"
- "Run all task flows in parallel for {task-name}"
- "Build everything for {task-name}"

If the developer says "implement {task-name}" without specifying a task flow and the task has 3+ task flows with independent subsets, suggest orchestrate-flow as the faster option.

## Hard preconditions

1. **task-design.md signed off** at `flow-storage/tasks/{task-name}/design/task-design.md`
2. **Task flow files exist** in `flow-storage/tasks/{task-name}/implement/flow-plan/` with `depends-on` frontmatter
3. **Git worktree support** (`git worktree --help` works on this machine)
4. **`orchestrate-implementer` subagent installed** at `.claude/agents/orchestrate-implementer.md` — verify with the Agent tool's `subagent_type` listing. If missing, the install is incomplete; run `instructions.md` Step 7.1 to re-install the Claude Code wrapper bundle.
5. **Working directory clean** (`git status` reports no changes). Phase 3 MERGE cherry-picks into the main WD; a dirty WD is unsafe. If dirty, offer Stash / Commit-first / Abort. See `.ai-workflow/flows/orchestrate-flow.md` Prerequisites for the script.

If any precondition fails, STOP and direct developer to design-flow, implement-flow (sequential), or `git stash` as appropriate.

## What this flow does

**4 phases, 2 gates.** Phases 2 and 4 auto-execute after the preceding gate is accepted.

### Phase 1: PLAN *(GATE TYPE B — STANDARD)*

- **1.1 LOAD** — Read all task flow files, design docs, KB context.
- **1.2 GROUP** — Topological sort by `depends-on` → compute dependency waves. Validate no two task flows in the same wave share files.
- **1.3 SCHEDULE** — Generate execution plan: waves, task flow groupings, intended subagent calls.
→ **Gate 1: Execution plan review** — developer adjusts grouping/reordering.

### Phase 2: EXECUTE *(auto — no gate)*

For each wave sequentially:

- **2.1 PREPARE WORKTREES** — **Skip the manual `git worktree add` loop on Claude Code.** The `orchestrate-implementer` subagent has `isolation: worktree` in its frontmatter, so Claude Code creates a fresh worktree under `.claude/worktrees/orchestrate-implementer-<id>/` for *each* subagent invocation automatically. You do not allocate worktree paths in advance and you do not run `git worktree add`. Just confirm `isolation: worktree` is set on the subagent (read `.claude/agents/orchestrate-implementer.md` if unsure).
- **2.2 LAUNCH SUBAGENTS** — **In one message, make N parallel Agent tool calls** (one per task flow in the current wave), all with `subagent_type: orchestrate-implementer`. Claude Code dispatches them concurrently and each gets its own auto-created worktree. The prompt to each subagent must contain:
  - The task name
  - The full path of the task flow file (e.g. `flow-storage/tasks/{task-name}/implement/flow-plan/{task-flow-file}.md`)
  - The wave number (informational)
  See `.ai-workflow/flows/orchestrate-flow.md` Phase 2.2 for the canonical sub-agent prompt.
- **2.3 MONITOR** — Each Agent call returns when its subagent completes. Collect the structured implementation report (parse it — the format is fixed; see `.claude/agents/orchestrate-implementer.md` § Report format). On `Status: failure`, surface the report's `Issues Encountered` to the developer and ask retry / skip / abort before starting the next wave.
- **2.4 WAIT FOR WAVE** — Because all N Agent calls were issued in the same message, control returns to you only when *all* of them have finished. That's your wave barrier — no separate wait logic needed. Display the wave summary before the next wave.

No blocking gates in this phase — the plan was already approved.

### Phase 3: MERGE *(GATE TYPE C — CRITICAL)*

- **3.1 COLLECT** — Aggregate the parsed reports (commit SHAs, files created/modified, test results, deviations) across all waves.
- **3.2 CHERRY-PICK** — Cherry-pick each subagent's commit SHA into the main branch in wave order. The subagents' worktree branches exist under whatever Claude Code named them (read each report's `Branch` field). Cherry-pick from the branch tip; the worktrees themselves are auto-cleaned by Claude Code when the parent message resolved, but the *commits* remain on their branches in the repository's reflog and refs until you reference or garbage-collect them. Resolve conflicts per the canonical flow.
- **3.3 VALIDATE** — Run full test suite, integration checks on the merged tree.
- **3.4 UPDATE RECORDS** — Transform all task flow files to knowledge records; update TDD, edge-cases, PATTERNS.
- **3.5 GENERATE REPORT** — Consolidated orchestration report.
→ **Gate 2: Final implementation review** — the ONE AND ONLY implementation gate. Developer reviews merged code, consolidated report, validation results.

### Phase 4: COMMIT *(auto — no gate)*

- **4.1 STAGE** — Stage all implementation files + updated knowledge records.
- **4.2 COMMIT** — `feat({task-name}): merged parallel implementation — {N} task flows in {W} waves`. **Worktree cleanup is automatic on Claude Code** — `isolation: worktree` reclaims worktrees when the subagent's parent message completes, so the `git worktree remove` loop from the canonical flow is a no-op here. The dangling sub-agent branches stay until manually pruned (`git branch -D orchestrate-implementer-*`) or garbage-collected. Opt-out: `--no-commit`.

## Hard rules

- Do not write any task code from this flow. Each task flow is implemented by an `orchestrate-implementer` subagent.
- Do not auto-accept any phase gate (Rule 6). Only 2 gates exist: Phase 1 (PLAN) and Phase 3 (MERGE).
- Subagents auto-proceed through implement-flow without gates (Rule 16 — sub-agent gate suppression).
- Each subagent works in its own auto-created worktree (Rule 17 — git worktree isolation; satisfied by `isolation: worktree`).
- Read `.ai-workflow/universal/rules.md` and `.ai-workflow/rules.md` before starting.

## Platform status: supported (preferred)

Claude Code is the reference platform for orchestrate-flow. The Agent tool natively supports parallel subagent invocation via multiple tool calls in one message ([sub-agents](https://code.claude.com/docs/en/sub-agents), [parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)), and the `isolation: worktree` frontmatter gives per-subagent CWD isolation without any manual `git worktree add` overhead.

If `.claude/agents/orchestrate-implementer.md` is missing or `isolation: worktree` was stripped from its frontmatter, fall back to the manual worktree path described in `.ai-workflow/flows/orchestrate-flow.md` § Platform Requirements.

For full instructions read `.ai-workflow/flows/orchestrate-flow.md`.
