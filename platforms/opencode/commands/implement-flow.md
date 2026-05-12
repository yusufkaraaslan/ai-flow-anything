---
description: Implement one task flow from a designed task — code, tests, doc-sync, commit
agent: build
---

Run the **implement-flow** for: $ARGUMENTS

The argument is one of:
- `<task>` — implement the next pending task flow for that task
- `<task> <task-flow-number-or-name>` — implement that specific task flow
- (empty) — ask the developer which task/task-flow

Load the `implement-flow` skill (or read `.ai-workflow/flows/implement-flow.md` directly) and execute its **2 phases**. Each phase contains sub-tasks that auto-proceed without intermediate gates:

### Phase 1: BUILD *(STANDARD gate)*
- **1.1 READ** — Load task flow file, task-design.md, task-technical-design.md sections, reference implementations.
- **1.2 PLAN** — Map acceptance criteria to existing patterns; surface ambiguities.
- **1.3 IMPLEMENT** — Write code per profile + project conventions.
- **1.4 AUTO-VALIDATE** — Run the project test suite (NEVER skip tests). If the test runner isn't available locally, STOP and tell the developer — do not auto-proceed.
- **1.5 DOC-SYNC** — Update task-technical-design.md with deviations from task-design.md (task-design.md itself stays immutable per Rule 9).
- → **Gate 1: Implementation review** — files changed, validation results, doc updates together.

### Phase 2: COMMIT *(CRITICAL gate)*
- **2.1 UPDATE TASK FLOW** — Mark task flow complete with `accepted-date`; append implementation notes.
- **2.2 COMMIT** *(executes only after the gate is accepted)* — Stage only this task flow's files; commit per conventional commits with task flow reference. Skip 2.2 if `--no-commit`.
- → **Gate 2: Commit review** — presented BEFORE 2.2 executes git commit.

Hard preconditions — **STOP** and refuse to proceed if any fail:
- `flow-storage/tasks/{task-name}/design/task-design.md` exists and is signed off (status: v2.0 (SIGNED OFF)). If not, tell the developer to run `/design-flow {task-name}` first.
- The target task flow file exists with `status: pending` or `status: in_progress`.
- The user's global rule "never skip any test" applies to sub-task 1.4 — failing tests do not auto-proceed; they surface at Gate 1.

If preconditions fail, do not improvise task code. Stop and ask the developer for direction.
