# Flow: Design

> **Profile:** Generic  
> **Purpose:** Design any task, system change, or update with visual diagrams

---

## When to Use

- Starting a new task
- Modifying existing functionality
- Refactoring or architecture changes
- Any work that needs planning before implementation

## Input

- Task idea or requirement (from developer)
- Existing project context (from knowledge base)

## Output

```
flow-storage/tasks/{task-name}/
├── design/
│   ├── task-design.md
│   ├── task-technical-design.md
│   ├── task-edge-cases.md
│   └── diagrams/
│       ├── 01-class-diagram.puml
│       ├── 02-package-diagram.puml
│       ├── 03-sequence-{flow}.puml (if needed)
│       └── 04-task-flow-dependencies.puml
└── implement/
    └── flow-plan/
        ├── task-flow-01-{descriptive-name}.md   # AI proposes based on task analysis
        ├── task-flow-02-{descriptive-name}.md
        └── ...                                  # Variable number (2-10+ depending on complexity)
```

## Prerequisites

- Project context loaded (`flow-storage/project/ARCHITECTURE.md`)
- Developer can describe the task

---

## Phases

This flow has **1 review gate** (CRITICAL) in Phase 1. Phase 2 auto-executes after acceptance — its only purpose is to lock the design and commit.

---

### Phase 1: DESIGN

**Purpose:** Load context, design visually, write spec docs, decompose into task flows — all as one reviewable package. The developer reviews everything at once at the gate.

**Sub-tasks (auto-proceed):**

**1.1 UNDERSTAND**
- Read `flow-storage/project/ARCHITECTURE.md`
- Read `flow-storage/project/CONVENTIONS.md`
- Read `flow-storage/project/PATTERNS.md`
- Search for related work in `flow-storage/tasks/`
- Explore codebase for similar implementations

**1.2 DESIGN (Visual-First)**
- Ask developer: "Describe this task in 1-2 sentences"
- Generate class diagram (PlantUML) showing main entities, relationships, key methods
- Generate package/module diagram showing where new code fits + dependencies
- Render every `.puml` to `.svg` (Rule 4 — diagrams mandatory means *visible* diagrams)
- If no renderer available locally, STOP — install per `universal/diagram-standards.md`, then resume

**1.3 SPECIFY**
- Create `task-design.md`: Overview, Goals/Non-Goals, Architecture (refs diagrams), Data Model, API, User Stories, Edge Cases, Open Questions
- Create `task-technical-design.md`: implementation approach, file structure, interface contracts, error handling, testing strategy
- Create `task-edge-cases.md`: boundary conditions, error scenarios, performance, security
- If task has complex flows, generate sequence diagram `diagrams/03-sequence-{flow}.puml` (rendered)

**1.4 PLAN**
- AI analyzes task and proposes task flow breakdown:
  - Names describe the WORK, not architectural layers (kebab-case)
  - NUMBER of task flows is flexible (2-10+ depending on complexity)
  - Examples:
    - Simple task: `data-model.md`, `component.md`, `tests.md` (3 task flows)
    - Complex task: `user-session-store.md`, `auth-service.md`, `login-form-component.md`, `api-error-handling.md`, `rate-limiter.md`, `tests.md` (6 task flows)
- Create task flow files in `implement/flow-plan/` with frontmatter:
  ```yaml
  ---
  task-flow: user-session-store
  task: "user-authentication"
  status: pending
  depends-on: []
  accepted-date: null
  tags: ["backend", "data-model"]
  ---
  ```
- Define acceptance criteria per task flow
- Build task flow dependency graph
- Generate `design/diagrams/04-task-flow-dependencies.puml` + render to `.svg`
- Run internal PLAN validation (surface failures at the gate):
  - Temporal contradictions in deps
  - Undefined types/symbols
  - Coverage gaps
  - Pairwise file overlap: any two task flows with NO depends-on edge between them but overlapping Files-to-Create/Modify lists. Add depends-on (force serial) or merge. Verifiable from file lists alone; orchestrate-flow's GROUP later enforces the same at wave granularity.
  - Oversized task flows (>1 day)
- Stage `flow-storage/tasks/{task-name}/` for preview
- Compose proposed commit message

**Artifacts (presented at gate):**
- `design/task-design.md`
- `design/task-technical-design.md`
- `design/task-edge-cases.md`
- `design/diagrams/01-class-diagram.puml` + `.svg`
- `design/diagrams/02-package-diagram.puml` + `.svg`
- `design/diagrams/03-sequence-{flow}.puml` + `.svg` (if needed)
- `design/diagrams/04-task-flow-dependencies.puml` + `.svg`
- Task flow files in `implement/flow-plan/task-flow-{NN}-{name}.md`
- Staged file list (`git diff --cached --stat`)
- Proposed commit message

**Gate:** Design + plan review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE C — CRITICAL)*

⚠️ **This is the ONLY gate.** Accepting locks task-design.md immutable per Rule 9 AND triggers Phase 2 (auto-executes FINALIZE + COMMIT). After this point, any change to task-design.md goes in task-technical-design.md instead.

**Failure modes to verify before [A]ccept:**
- PLAN integrity (any internal check failures from sub-task 1.4 still unresolved?)
- Anything you'd want to change in task-design.md? Now is the only time.
- Diagrams cross-referenced correctly from task-design.md?
- All diagrams rendered: every `.puml` has a sibling `.svg` or `.png`
- All task flows from 1.4 actually written to disk?
- Files staged: only `flow-storage/tasks/{task-name}/` contents — no stray edits?
- Commit message: task name correct? Task flow count matches `flow-plan/`? Summary captures *why* not *what*?

**Opt-out:** `--no-commit` skips Phase 2's commit sub-task. FINALIZE (sign-off + lock) still runs regardless — the design is always locked at this point per Rule 9.

---

### Phase 2: LOCK & COMMIT

**Purpose:** Mechanically apply the decisions already reviewed and accepted at Phase 1. No separate gate — the scope was already approved.

**Sub-tasks (auto-execute after Phase 1 [A]ccept):**

**2.1 FINALIZE**
- Update task-design.md status header:
  ```markdown
  > **Status:** v2.0 (SIGNED OFF)
  > **Signed Off By:** {developer}
  > **Date:** {date}
  > **Immutable:** Yes
  ```

**2.2 COMMIT** *(skipped with --no-commit)*
- Run `git commit` with the message previewed at Phase 1:
  ```
  design({task-name}): sign off task-design.md and {N} task flows

  {one-line summary of task scope}

  Task flows created: {list, comma-separated}
  Diagrams: class, package, sequence (and any others)
  ```

---

## Visual-First Design Protocol

```
Developer: "I want a user authentication task"

AI: Generating class diagram v1...
    [diagram: User, AuthService, TokenManager, UserRepository]

AI: Does this match your intent?
    [A]ccept / [M]odify / [R]estart

Developer: M
Developer: "Add PasswordValidator and EmailService"

AI: Updated diagram v2...
    [diagram with added classes]

AI: [A]ccept / [M]odify / [R]estart

Developer: A

AI: Generating text documentation from approved diagram...
```

---

## Customization

Users can customize this flow by editing the generated copy in `.ai-workflow/flows/`:

- Add sub-tasks within Phase 1 (e.g., "Security Review" before the gate)
- Remove sub-tasks (e.g., skip sequence diagrams for simple tasks)
- Change diagram format (Mermaid instead of PlantUML)
- Add task flow types (e.g., "Migration", "Config")

⚠️ Avoid adding gates without a clear reason — the single-gate cadence is deliberate. Each new gate is a new user-facing interruption.