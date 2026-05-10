# Flow: Implement

> **Profile:** Generic  
> **Purpose:** Implement task flows one at a time with validation and review

---

## When to Use

After the design flow has completed and design is signed off.

## Input

- Task flow files from `flow-storage/tasks/{task-name}/implement/flow-plan/`
- task-design.md and task-technical-design.md
- Prior completed task flows (for context)

## Output

- Implemented source code
- Updated task flow files (transformed to knowledge records)
- Tests
- Updated task-technical-design.md with deviations

## Prerequisites

- Design flow completed and signed off
- At least one task flow file exists

---

## Phases

This flow has **2 phases, 2 gates**. Each phase contains multiple sub-tasks that auto-proceed without intermediate gates. See `universal/workflow-structure.md` for the canonical pattern.

---

### Phase 1: BUILD

**Purpose:** Load context, plan, write code, validate, and sync docs as one reviewable package.

**Sub-tasks (auto-proceed):**

**1.1 READ**
- Read `flow-storage/project/{ARCHITECTURE,CONVENTIONS,PATTERNS}.md`
- Read the task flow file: extract acceptance criteria, files to create/modify, dependencies
- Read design docs: relevant sections of `task-design.md`, `task-technical-design.md`, `task-edge-cases.md`
- Read completed prior task flows (status: complete) and their implementation notes
- Find 1-2 reference implementations in codebase
- **Hard refusal** if no `task-design.md` or it is not signed off — point developer at design-flow

**1.2 PLAN**
- Analyze each acceptance criterion: code needed, patterns to apply, edge cases
- Map to existing patterns in `PATTERNS.md` and reference implementations
- Structure: files to create/modify, order of implementation, test approach

**1.3 IMPLEMENT**
- Write code incrementally: domain/model layer → business logic → integration/API layer
- Follow naming from `CONVENTIONS.md`, patterns from `PATTERNS.md`, architecture from `ARCHITECTURE.md`
- Handle edge cases from `task-edge-cases.md`
- Cite reference implementations only when non-obvious

**1.4 AUTO-VALIDATE**
- Run architecture/style checks specified in profile `rules.md`
- Run the project's test suite (NEVER skip tests — user global rule)
- Check integration points against task-design.md signal flow
- Check for syntax errors, naming convention violations, TODO/FIXME, hardcoded values, duplicated code, missing error handling
- Verify all acceptance criteria addressed
- If validation fails, do **not** auto-proceed — surface failures at the Phase 1 gate so developer can [F]eedback to fix

**1.5 DOC-SYNC**
- Update `task-technical-design.md` with deviations from `task-design.md` (Rule 9 — DESIGN stays immutable; deviations live in TDD)
- Update `task-edge-cases.md` with new discoveries
- Update `flow-storage/project/PATTERNS.md` if a new reusable pattern emerged

**Artifacts (presented at gate):**
- Files changed (created + modified)
- Test results + coverage delta
- Validation report
- Doc updates (task-technical-design.md, task-edge-cases.md, PATTERNS.md)

**Gate:** Implementation review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

---

### Phase 2: COMMIT

**Purpose:** Transform the task flow file into a knowledge record and commit the work to git as one atomic unit.

**Sub-tasks (auto-proceed):**

**2.1 UPDATE TASK FLOW**
- Update task flow frontmatter:
  ```yaml
  status: complete
  accepted-date: {date}
  ```
- Append implementation notes: what was implemented, patterns used, deviations from design, testing approach
- Add code references (file paths, key functions/classes)

**2.2 COMMIT** *(executes only after Phase 2 gate is accepted)*
- Stage only the files this task flow produced
- Compose commit message (conventional commits):
  ```
  feat({task-name}): {task-flow-name} — brief description

  - Change 1
  - Change 2

  Refs: {task-flow-name}
  ```
- Run `git commit`

**Artifacts (presented at gate):**
- Knowledge record (the updated task flow file)
- Staged file list (`git diff --cached --stat`)
- Proposed commit message

**Gate:** Commit review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE C — CRITICAL)*

⚠️ Commit lands in history; amending later is friction.

**Failure modes to verify before [A]ccept:**
- Files staged: any unrelated stray edits sneaking in?
- Commit message: scope/type accurate? Description focuses on *why* not *what*?
- Co-author trailer present if required by project policy?
- Hooks won't fail (e.g. linter clean, no test failures)?
- Pre-existing failing tests not silently bypassed?

**Opt-out:** `--no-commit` skips sub-task 2.2 (gate still presents).

---

## Resume Logic

Before starting, check for existing progress:

1. Read all task flow files in `flow-storage/tasks/{task-name}/implement/flow-plan/`
2. Find first task flow with `status: pending` or `status: in_progress`
3. If found, ask: "Resume from {task-flow-name}?"
4. If yes, skip completed task flows

**Status display:**
```
Implementation Status: {task-name}

  01-user-session-store.md    complete   (2026-04-23)
  02-auth-service.md          in_progress
  03-login-form.md            pending    (blocked by: auth-service)
  04-api-integration.md       pending    (blocked by: auth-service)
  05-tests.md                 pending    (blocked by: all above)

Progress: 1/5 complete
Next: 02-auth-service.md
```

---

## Customization

Users can customize:
- Add validation steps inside Phase 1 sub-task 1.4 AUTO-VALIDATE
- Change commit message format in Phase 2 sub-task 2.2 COMMIT
- Add deployment step after commit
- Add notification step (Slack, email)

---

## Sub-Agent Mode

When this flow is invoked as a **sub-agent** by the orchestrate-flow, it runs in a different mode:

**Trigger:** The orchestrator passes `mode: sub-agent` in its launch prompt.

**Behavior changes:**
- **No `[A]/[F]/[R]` gates.** All sub-tasks auto-proceed without user-facing interruption. The gates are suppressed — the orchestrator handles the only gate (Phase 3 MERGE).
- **Works in an isolated git worktree** at `.ai-workflow/worktrees/{task-name}/{task-flow}/`. Do not modify files outside this worktree.
- **Commits in the worktree** at the end of Phase 2 (standard conventional commit per Rule 7). Do not push.
- **Returns a structured implementation report** instead of presenting gate artifacts. Format:

```markdown
## Sub-Agent Report: {task-flow} ({task-name})

**Status:** success | failure
**Worktree:** .ai-workflow/worktrees/{task-name}/{task-flow}/
**Commit SHA:** <sha>

### Files Created
- path/to/file1
- path/to/file2

### Files Modified
- path/to/file3

### Test Results
- {passing}/{total} passing
- Coverage delta: +X%

### Deviations from Design
- {deviation 1}
- {deviation 2}

### Issues Encountered
- {issue 1}
- {issue 2}
```

**How the orchestrator calls this flow as a sub-agent:**
```
You are a sub-agent of the implement-orchestrator for task "{task-name}".
Your job: implement ONE task flow using implement-flow in sub-agent mode.

Task flow file:
  flow-storage/tasks/{task-name}/implement/flow-plan/{task-flow-file}.md

Mode: sub-agent
  - Follow the implement-flow phases (READ → PLAN → IMPLEMENT →
    AUTO-VALIDATE → DOC-SYNC → UPDATE TASK FLOW → COMMIT).
  - Auto-proceed through ALL sub-tasks. Do NOT present [A]/[F]/[R] gates.
  - Work in your git worktree: .ai-workflow/worktrees/{task-name}/{task-flow}/
  - Commit in your worktree when done.
  - Return the structured implementation report when complete.
```
