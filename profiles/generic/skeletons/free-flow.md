# Flow: Free Flow

> **Profile:** Generic  
> **Purpose:** Handle quick, unstructured work — bug fixes, small enhancements, one-line design tweaks, minor refactors — without the full design-implement-review chain

---

## When to Use

- Bug fixes and patches
- Small enhancements or design tweaks
- Minor refactors (rename, extract, reorder)
- Any work that is too small to warrant a full design-flow + implement-flow cycle
- Developer says "quick fix," "just change X," "small bug," "tweak Y," "hotfix"

## When NOT to Use

- New features requiring design — use **design-flow** then **implement-flow**
- Multi-day work with task flow decomposition — use **design-flow** + **parallel-implement-flow**
- The developer explicitly wants the full design-implement-review chain

**Rule of thumb:** If you'd need to create a class diagram to explain the work, use design-flow instead. If the change can be described in 2-3 sentences, free-flow is the right tool.

## Input

- Developer describes the issue or change (natural language)
- Existing project context (from knowledge base)

## Output

- Fixed/changed source code
- A task-flow file recording the work (one of two places, see below)
- Updated impacted task-flow records (if any)
- Updated task-technical-design.md deviations (if touching a designed task)
- Updated `flow-storage/project/PATTERNS.md` (if a new pattern emerged)

### KB placement

| Case | Task-flow file written to |
|------|--------------------------|
| **Related to existing task** | `flow-storage/tasks/{task-name}/implement/flow-plan/task-flow-{free-flow-name}.md` |
| **Standalone** (no related task) | `flow-storage/tasks/{free-flow-name}/implement/flow-plan/task-flow-{free-flow-name}.md` |

When related to an existing task, the free-flow file lives alongside the task flows planned by design-flow — it is a late injection into the plan. When standalone, the directory has just `implement/flow-plan/` (no `design/`, no diagrams).

## Prerequisites

- Project context loaded (`flow-storage/project/ARCHITECTURE.md`)
- Developer can describe the change
- The change is small enough to discuss in one session

---

## Phases

This flow has **3 phases, 3 gates** — TALK → PLAN → FIX. Each phase contains multiple sub-tasks that auto-proceed without intermediate gates.

---

### Phase 1: TALK

**Purpose:** Understand what the developer needs, load context, and determine the scope and KB impact of the change.

**Sub-tasks (auto-proceed):**

**1.1 ASK**
- Ask the developer to describe the issue or change in 2-3 sentences
- Clarify: is this a bug fix, enhancement, refactor, design tweak, or something else?
- Ask: "Is this related to an existing task in the flow database, or is it standalone?"
- If the developer doesn't know, proceed to 1.2 and figure it out

**1.2 CONTEXT**
- Load `flow-storage/project/{ARCHITECTURE,CONVENTIONS,PATTERNS}.md`
- Load `flow-storage/project/DECISIONS.md` (relevant ADRs)
- Search `flow-storage/tasks/` for related work:
  - Search task-design.md files for keywords matching the described change
  - Search task-flow files in `implement/flow-plan/` for file names or features that overlap
  - Look for existing free-flow records under tasks (task-flow files with `free-flow: true` frontmatter)
  - Read `flow-storage/tasks/*/docs/lessons-learned.md` for relevant prior issues

**1.3 SCOPE**
- Determine the scope category:
  - **Bug fix:** restore intended behavior
  - **Enhancement:** add small feature or improve existing
  - **Design tweak:** change behavior/API/interface of an existing feature
  - **Refactor:** restructure code without changing behavior
  - **Other:** described by developer
- Determine KB placement:
  - If the change touches files belonging to an existing task (found in 1.2), it is **related** — use that task's `flow-storage/tasks/{task-name}/implement/flow-plan/` directory
  - If no existing task covers this area, it is **standalone** — create a new directory
- If ambiguous, present findings and let the developer decide

**Artifacts (presented at gate):**
- Issue summary (what the developer described)
- Scope category (bug fix / enhancement / design tweak / refactor / other)
- KB placement decision: related to task X (linked) OR standalone (new directory)
- Related KB entries found (linked task designs, task-flow files, lessons learned)

**Gate:** Scope review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

---

### Phase 2: PLAN

**Purpose:** Analyze the change, detect impacts on existing flow database entries, and propose a concrete fix plan.

**Sub-tasks (auto-proceed):**

**2.1 ANALYZE**
- Identify files to touch (create, modify, or delete)
- Find 1-2 reference implementations in the codebase for similar patterns
- Read the relevant source code to understand the current state
- Determine the minimal change that addresses the issue (Rule 14: backward compatibility)

**2.2 CROSS-REF** *(critical — KB impact detection)*
- For each file identified in 2.1, search all task-flow files in `flow-storage/tasks/*/implement/flow-plan/` for matches in their "Files to Create" and "Files to Modify" sections
- Any task-flow file that declares the same file path in "Files to Create" or "Files to Modify" is **impacted** — its record needs updating after the fix
- Also check: does the change alter behavior described in any signed-off `task-design.md`? If yes, flag for a TDD deviation entry
- Build an impact report listing:
  - **Impacted task-flow files** (with paths) — will need "Free-Flow Update" appended to Implementation Notes
  - **Impacted task-design.md files** — will need a deviation entry in their paired `task-technical-design.md`
  - **No impact** — no existing KB entries touched

**2.3 PROPOSE**
- Write a concise plan: files to change, what changes in each, test strategy
- If the change is a bug, include: root cause, fix, regression risk
- Present the impact report from 2.2 alongside the plan
- Determine the free-flow name (kebab-case, descriptive of the work, e.g. `fix-login-timeout`, `add-search-debounce`, `refactor-user-validator`)

**Artifacts (presented at gate):**
- Fix/enhancement plan (files, changes per file, test approach)
- Impact report: KB entries needing update (task-flow files, TDD deviations)
- Free-flow name for the new task-flow record
- If standalone: proposed directory name for `flow-storage/tasks/{free-flow-name}/`

**Gate:** Plan review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

---

### Phase 3: FIX

**Purpose:** Implement the change, validate it, record everything in the knowledge base, and commit.

**Sub-tasks (auto-proceed):**

**3.1 IMPLEMENT**
- **First, create a minimal task-flow stub** at the KB location determined in Phase 1.3. Write it with `status: in_progress` frontmatter so an interrupted session can be resumed:
  ```yaml
  ---
  task-flow: {free-flow-name}
  task: "{task-name-or-free-flow-name}"
  status: in_progress
  depends-on: []
  accepted-date: null
  tags: ["free-flow", "{scope-category}"]
  free-flow: true
  summary: "{one-line description}"
  ---
  ```
  (Phase 3.3 will fill in the full record and update status to `complete`.)
- Write the code changes per the plan from Phase 2
- Follow patterns from `CONVENTIONS.md` and `PATTERNS.md`
- Apply profile rules from the project's active profile
- Handle edge cases surfaced during TALK or PLAN
- If the change is a bug fix, include a regression test

**3.2 VALIDATE**
- Run the project's test suite (NEVER skip tests — user global rule)
- Run lint/typecheck as specified in profile rules
- Verify the fix addresses the issue described in Phase 1
- Check backward compatibility (Rule 14): do existing tests still pass? Are integration points intact?
- If validation fails, fix and re-validate before proceeding

**3.3 RECORD**
- Update the task-flow stub (created in 3.1) with the full record:
  - Change frontmatter `status` from `in_progress` to `complete`, set `accepted-date`
  - Add the committed content (sections 1-5 below)
  ```yaml
  ---
  task-flow: {free-flow-name}
  task: "{task-name-or-free-flow-name}"
  status: complete
  depends-on: []
  accepted-date: {date}
  tags: ["free-flow", "{scope-category}"]
  free-flow: true
  summary: "{one-line description of the change}"
  ---
  ```
- Populate the task-flow file with:
  1. **Summary** — What was done and why (the summary from frontmatter, expanded)
  2. **Scope** — Bug fix / enhancement / design tweak / refactor / other
  3. **Files Changed** — Created and modified, with brief per-file notes
  4. **Implementation Notes** — Key decisions, patterns used
  5. **Root Cause** (bug fixes only) — What was wrong and why the fix works
- Apply KB updates from the Phase 2 impact report:
  - For each **impacted task-flow file**: append a `### Free-Flow Update ({free-flow-name})` section to its Implementation Notes that describes what changed.
  - For each **impacted task-design.md**: add a deviation entry to the paired `task-technical-design.md` under the "Deviations from task-design.md" section with a backlink to the free-flow task-flow file.
  - If a new reusable pattern emerged, add it to `flow-storage/project/PATTERNS.md`.
  - If a notable architectural decision was made, add an entry to `flow-storage/project/DECISIONS.md`.

**3.4 COMMIT**
- Stage the files changed + the new task-flow file + any updated KB files (impacted task-flows, TDD, PATTERNS.md, DECISIONS.md)
- Compose commit message (conventional commits):
  ```
  {type}({task-name-or-scope}): {free-flow-name} — {one-line summary}

  Scope: {bug fix / enhancement / design tweak / refactor / other}
  KB: {free-flow record + N impacted task-flows updated}

  Refs: task-flow-{free-flow-name}
  ```
  Where `{type}` is `fix` for bugs, `feat` for enhancements, `refactor` for refactors, or `chore` for other.
- Run `git commit`

**Artifacts (presented at gate):**
- Files changed (created + modified) with `git diff --cached --stat`
- Test results + validation report
- The new task-flow file (showing full content)
- KB updates applied (impacted task-flow files with Free-Flow Update sections, TDD deviations added, PATTERNS/DECISIONS entries)
- Proposed commit message

**Gate:** Fix + KB review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE C — CRITICAL)*

⚠️ Commit lands in history; KB updates become part of the permanent record.

**Failure modes to verify before [A]ccept:**
- Code change addresses the issue described in Phase 1 TALK?
- Regression test included (if bug fix)?
- Full test suite passes (no regressions)?
- KB impact report from Phase 2 fully applied — all impacted task-flows updated, all TDD deviations recorded?
- Task-flow file frontmatter correct: `free-flow: true`, `tags` include the scope category?
- Staged files: only the intended changes — no stray edits from unrelated work?
- Commit message: type correct (`fix`/`feat`/`refactor`/`chore`)? KB impact mentioned?

**Opt-out:** `--no-commit` skips sub-task 3.4 entirely (gate still presents — code and KB record remain unstaged, uncommitted).

---

## Sub-Agent Mode

When this flow is invoked as a **sub-agent** by an orchestrator or other flow, it runs in sub-agent mode (Rule 16):

- **No `[A]/[F]/[R]` gates.** All sub-tasks auto-proceed. The caller handles the gate.
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

---

## Resume Logic

Free-flow is designed for single-session work. If interrupted:

1. Check for partially written task-flow files with `free-flow: true` and `status: in_progress` in `flow-storage/tasks/*/implement/flow-plan/`
2. If found, ask: "Resume free-flow: {free-flow-name}?"
3. Restart from Phase 1.3 SCOPE (re-load context, then pick up where left off)

---

## Customization

- Add validation steps inside Phase 3 sub-task 3.2 for custom checks
- Change commit message format in Phase 3 sub-task 3.4
- Add notification step after commit (Slack, email)
- Override the scope categories with project-specific ones in `.ai-workflow/rules.md`
