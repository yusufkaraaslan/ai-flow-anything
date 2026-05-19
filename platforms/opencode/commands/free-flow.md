---
description: Quick fix, bug fix, tweak, hotfix, small change, or minor refactor — 3-phase TALK → PLAN → FIX
agent: build
---

Run the **free-flow** for: $ARGUMENTS

The argument is one of:
- `<issue-description>` — describe the bug, enhancement, tweak, or refactor
- (empty) — ask the developer what needs fixing

Load the `free-flow` skill (or read `.ai-workflow/flows/free-flow.md` directly) and execute its **3 phases**. Each phase contains sub-tasks that auto-proceed without intermediate gates:

### Phase 1: TALK *(STANDARD gate)*
- **1.1 ASK** — Clarify: bug fix / enhancement / design tweak / refactor? Related to existing task?
- **1.2 CONTEXT** — Load project KB; search `flow-storage/tasks/` for related task designs, task-flow files, prior free-flow records, lessons learned.
- **1.3 SCOPE** — Categorize; determine KB placement (related → nest under task; standalone → new directory).
- → **Gate 1: Scope review** — issue summary, scope category, KB placement decision.

### Phase 2: PLAN *(STANDARD gate)*
- **2.1 ANALYZE** — Identify files to touch; find reference implementations; read relevant source.
- **2.2 CROSS-REF** — For each file, search all task-flow files' "Files to Create" / "Files to Modify" for overlaps. Build KB impact report.
- **2.3 PROPOSE** — Concise plan (files, changes, test strategy). Bugs: root cause + regression risk.
- → **Gate 2: Plan review** — plan, impact report, free-flow name.

### Phase 3: FIX *(CRITICAL gate)*
- **3.1 IMPLEMENT** — Create task-flow stub (`status: in_progress`); write code per plan; follow project conventions.
- **3.2 VALIDATE** — Run test suite (NEVER skip tests); lint/typecheck; check backward compatibility (Rule 14).
- **3.3 RECORD** — Update stub to complete; apply KB impact updates (append "Free-Flow Update" to impacted task-flows, TDD deviations, PATTERNS.md/DECISIONS.md).
- **3.4 COMMIT** *(executes only after the gate is accepted)* — Stage code + KB records; conventional commit; `git commit`. Skip 3.4 if `--no-commit`.
- → **Gate 3: Fix + KB review** — presented BEFORE 3.4 executes git commit.

Hard preconditions — none beyond project context loaded. Free-flow is designed for work with no prior design.
