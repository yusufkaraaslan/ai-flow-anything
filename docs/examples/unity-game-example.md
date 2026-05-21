# Example: Unity Game Workflow

Walkthrough of using ai-flow-anything with a Unity project.

---

## Project Setup

**Project:** `SpaceDefender` — Unity 2D space shooter

**Structure:**
```
SpaceDefender/
├── Assets/
│   ├── Scripts/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   └── Views/
│   └── Scenes/
├── ProjectSettings/
└── Packages/
```

---

## Step 1: Initialize

```
Developer: "Initialize ai-flow-anything for this project"

AI: Detecting project type...
    Found: Assets/, *.cs, ProjectSettings/
    Confidence: 100% — Unity project

AI: Loading Unity Game profile...

AI: Running discovery...
    Project: SpaceDefender
    Module: Assets/Scripts/
    Namespace: SpaceDefender
    DI: None detected
    Async: Coroutines
    Features: Controllers, Models, Views

AI: Generating flows...
    ✓ design-flow.md
    ✓ implement-flow.md
    ✓ free-flow.md
    ✓ parallel-implement-flow.md
    ✓ pr-flow.md
    ✓ test-flow.md
    ✓ deploy-flow.md
    ✓ docs-flow.md

AI: Install complete!
    Flows available in .ai-workflow/flows/
```

---

## Step 2: Design New Task

**Task:** Shield Power-Up

```
Developer: "Design a task called shield-powerup"

AI: Phase 1: SPECIFY
    [1.1 UNDERSTAND] Loading SpaceDefender architecture...
                     Found similar tasks: SpeedBoost, DoubleShot
                     Reading reference implementations...

    [1.2 DESIGN]     Class Diagram v1:
                     [ShieldPowerUp] ---extends---> [PowerUpBase]
                     [ShieldPowerUp] ---uses---> [ShieldController]
                     [ShieldController] ---uses---> [PlayerModel]

                     Developer: Add ShieldView component

                     Class Diagram v2 (rendered to .svg):
                     [ShieldPowerUp] ---extends---> [PowerUpBase]
                     [ShieldPowerUp] ---uses---> [ShieldController]
                     [ShieldController] ---uses---> [PlayerModel, ShieldView]

    [1.3 SPECIFY]    Creating task-design.md...
                     Creating task-technical-design.md...
                     Creating task-edge-cases.md...

→ Gate 1: Design package review — diagrams + spec docs together
    Developer: Accept

AI: Phase 2: COMMIT
    [2.1 PLAN]       Task flows:
                     1. shield-data-model.md — Shield data
                     2. shield-controller.md — Shield logic
                     3. shield-view.md — Shield visual
                     4. shield-tests.md — Unit + PlayMode tests
                     5. shield-integration.md — Scene setup, prefab

    [2.2 FINALIZE]   Signing off task-design.md (immutable)...

→ Gate 2: Plan + final approval — presented BEFORE git commit
    Developer: Accept

    [2.3 COMMIT]     git commit -m "design(shield-powerup): sign off
                     task-design.md and 5 task flows"
                     Handing off to implement-flow...
```

---

## Step 3: Implement

```
Developer: "Implement the next task flow for shield-powerup"

AI: Resuming from first pending task flow...
    Task flow: task-flow-shield-data-model.md

AI: Phase 1: BUILD
    [1.1 READ]         Loading context...
                       Reading ShieldPowerUp design...
                       Reference: SpeedBoost implementation
    [1.2 PLAN]         Files to create:
                       - Assets/Scripts/Models/ShieldModel.cs
                       - Assets/Scripts/Controllers/ShieldController.cs
    [1.3 IMPLEMENT]    Creating ShieldModel.cs...
                       Creating ShieldController.cs...
    [1.4 AUTO-VALIDATE] ✓ No UnityEngine in Model
                       ✓ Controller has lifecycle methods
                       ✓ Events unsubscribed in OnDisable
                       ✓ EditMode tests pass (12/12)
    [1.5 DOC-SYNC]     Updated task-technical-design.md with deviation
                       (added IShieldController interface)

→ Gate 1: Implementation review — files changed + validation + doc updates
    Developer: Accept

AI: Phase 2: COMMIT
    [2.1 UPDATE TASK FLOW] task-flow-shield-data-model.md → status: complete

→ Gate 2: Commit review — presented BEFORE git commit
    Developer: Accept

    [2.2 COMMIT]       git commit -m "feat(shield-powerup): shield data model"

AI: Next task flow: task-flow-shield-controller.md
    
[Repeat for all task flows...]
```

---

## Step 4: PR

```
Developer: "Validate shield-powerup for PR"

AI: Mode: pre-pr
    
    Step 1: Task flows complete ✓
    Step 2: Documentation ✓
    Step 3: Architecture ✓
    Step 4: Performance ✓
    Step 5: Tests ✓ (12/12 passing)
    Step 6: Integration ✓
    
    Status: READY FOR PR
    
Developer: [Creates PR]

[Reviewers comment...]

Developer: "Process PR feedback for shield-powerup" (PR flow, feedback mode)

AI: Processing feedback...
    Comment 1: "Shield duration should be configurable"
    → Created feedback-1.md
    → Suggested fix: Add [SerializeField] duration
    Developer: Yes
    
    Comment 2: "Missing null check in controller"
    → Created feedback-2.md
    → Suggested fix: Add null check
    Developer: Yes
    
AI: Committing fixes...
    fix(shield-powerup): address PR feedback
    
[PR merged...]

Developer: "Capture lessons from shield-powerup" (PR flow, capture mode)

AI: Capturing knowledge...
    ✓ lessons-learned.md created
    ✓ PATTERNS.md updated (Shield pattern)
    ✓ CONVENTIONS.md updated (null checks)
    
    Task complete!
```

---

## Result

**Knowledge Base:**
```
flow-storage/tasks/shield-powerup/
├── design/
│   ├── task-design.md (signed off)
│   ├── task-technical-design.md
│   ├── task-edge-cases.md
│   └── diagrams/
│       ├── 01-class-diagram.puml
│       ├── 02-package-diagram.puml
│       └── 04-task-flow-dependencies.puml
├── implement/
│   └── flow-plan/
│       ├── task-flow-01-model.md (knowledge record)
│       ├── task-flow-02-controller.md (knowledge record)
│       ├── task-flow-03-view.md (knowledge record)
│       ├── task-flow-04-tests.md (knowledge record)
│       └── task-flow-05-integration.md (knowledge record)
├── pr/
│   └── feedback/
│       ├── feedback-1-review.md
│       └── feedback-2-review.md
└── docs/
    └── lessons-learned.md
```

**Project Knowledge Updated:**
- `flow-storage/project/PATTERNS.md` — Shield pattern added
- `flow-storage/project/CONVENTIONS.md` — Null check convention added

---

## Lessons

1. **Visual-first design caught missing component early.**
2. **Per-task-flow commits kept history clean.**
3. **Feedback task flows captured review knowledge permanently.**
4. **Patterns catalog grew — next power-up will be faster.**
