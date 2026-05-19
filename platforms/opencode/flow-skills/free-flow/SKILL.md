---
name: free-flow
description: Use when the developer says "quick fix X", "bug fix", "fix X", "tweak Y", "hotfix", "small change", "patch X", "refactor X", "just change X", or asks for a quick unstructured change. Handles bug fixes, small enhancements, design tweaks, and minor refactors — work too small for the full design-implement-review chain — in 3 phases with 3 review gates [A]/[F]/[R] (TALK → PLAN → FIX). Checks the flow database for impacted task-flow records and updates them. See .ai-workflow/flows/free-flow.md for the full canonical flow. Also supports sub-agent mode: gates are suppressed and the flow auto-proceeds, returning a structured report.
license: MIT
metadata:
  audience: developers
  workflow: task-development
  phase: free
---

# free-flow (OpenCode wrapper)

This skill runs the **free flow** — a lightweight alternative to the design-implement-review chain for quick, unstructured work. The full flow lives in `.ai-workflow/flows/free-flow.md` (or the canonical `profiles/generic/skeletons/free-flow.md` if the project uses the defer pattern). Read it and execute.

## When to invoke

**User-direct mode (with gates):**
- "Quick fix: {describe issue}"
- "Fix the {bug}"
- "Tweak {thing}"
- "Small change: {describe}"
- "Refactor {thing}"
- "Hotfix for {issue}"
- "Just change {X} to {Y}"
- Any request where the developer signals this is a small, unstructured change

**Sub-agent mode (no gates):**
- Check if the launch context includes `mode: sub-agent`.
- If yes: auto-proceed through all sub-tasks, suppress all `[A]/[F]/[R]` gates, and return a structured free-flow report per Rule 16.

## Hard preconditions

None — free-flow is designed for work that has no prior design. It discovers context and KB impact at runtime.

## What this flow does (user-direct mode)

**3 phases, 3 gates.** Each phase contains multiple sub-tasks that auto-proceed without intermediate gates. See `.ai-workflow/universal/workflow-structure.md` "Phase Architecture" → "Free Flow Phases".

### Phase 1: TALK *(GATE TYPE B — STANDARD)*

Sub-tasks (auto-proceed):
- **1.1 ASK** — Clarify scope: bug fix, enhancement, design tweak, refactor, or other? Related to an existing task?
- **1.2 CONTEXT** — Load project KB; search `flow-storage/tasks/` for related task designs, task-flow files, prior free-flow records, lessons learned.
- **1.3 SCOPE** — Categorize and determine KB placement (related to existing task → nests under its `implement/flow-plan/`; standalone → creates new directory).

→ **Gate 1: Scope review** — developer sees issue summary, scope category, KB placement decision.

### Phase 2: PLAN *(GATE TYPE B — STANDARD)*

Sub-tasks (auto-proceed):
- **2.1 ANALYZE** — Identify files to touch; find reference implementations; read relevant source.
- **2.2 CROSS-REF** — For each file, search all task-flow files' "Files to Create" and "Files to Modify" sections for overlaps. Build an impact report of KB entries needing updates.
- **2.3 PROPOSE** — Write a concise plan (files, changes, test strategy). Bug fixes include root cause + regression risk.

→ **Gate 2: Plan review** — developer sees the plan, KB impact report, and free-flow name.

### Phase 3: FIX *(GATE TYPE C — CRITICAL)*

Sub-tasks (auto-proceed):
- **3.1 IMPLEMENT** — Create a minimal task-flow stub (`status: in_progress`); write code per the plan; follow project conventions.
- **3.2 VALIDATE** — Run the project test suite (NEVER skip tests); lint/typecheck; verify fix addresses the issue; check backward compatibility (Rule 14).
- **3.3 RECORD** — Update the stub to full record (`status: complete`); apply KB impact updates: append "Free-Flow Update" to impacted task-flows, add TDD deviations for impacted designs, update PATTERNS.md/DECISIONS.md if patterns or decisions emerged.
- **3.4 COMMIT** *(executes only after the gate is accepted)* — Stage code + task-flow record + KB updates; conventional commit (`fix`/`feat`/`refactor`/`chore`); `git commit`.

→ **Gate 3: Fix + KB review** — presented BEFORE 3.4 executes git commit. Opt-out: `--no-commit` skips sub-task 3.4 entirely.

## Sub-agent mode

When invoked by a parent flow (`mode: sub-agent` in the launch context):

- **No `[A]/[F]/[R]` gates** — all sub-tasks auto-proceed (Rule 16).
- **Returns a structured report** instead of presenting gate artifacts:

```markdown
## Free-Flow Sub-Agent Report: {free-flow-name}

**Status:** success | failure
**Scope:** {bug fix / enhancement / design tweak / refactor / other}
**Commit SHA:** <sha>

### Files Changed
- path/to/file1 (created/modified)
- path/to/file2 (created/modified)

### KB Updates
- Task-flow record: flow-storage/tasks/{task-name}/implement/flow-plan/task-flow-{free-flow-name}.md
- Impacted task-flows updated: {count} ({list})
- TDD deviations: {count} ({list})
- PATTERNS.md: {entry if added}
- DECISIONS.md: {entry if added}

### Test Results
- {passing}/{total} passing
- Coverage delta: +X%

### Issues Encountered
- {issue or "none"}
```

For full instructions read `.ai-workflow/flows/free-flow.md`.
