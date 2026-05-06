# Knowledge Base Specification

> **Applies to:** ALL profiles  
> **Purpose:** Define the three-tier knowledge base structure and maintenance rules

---

## Three-Tier Knowledge Base

```
┌─────────────────────────────────────────┐
│           Team-Level KB                 │
│   flow-storage/team/                            │
│   - Onboarding, workflows, glossary     │
├─────────────────────────────────────────┤
│          Project-Level KB               │
│   flow-storage/project/                         │
│   - Architecture, conventions, patterns │
├─────────────────────────────────────────┤
│          Task-Level KB               │
│   flow-storage/tasks/{task-name}/                 │
│   - Design, technical design, edge cases, task flows     │
└─────────────────────────────────────────┘
```

---

## Tier 1: Task-Level Knowledge Base

Created for every task. Grows as the task is designed, implemented, and reviewed. Each flow owns its own subfolder under the task root.

### Directory Structure

```
flow-storage/tasks/{task-name}/
├── design/                                       # design-flow output
│   ├── task-design.md                            # Design document (immutable after sign-off)
│   ├── task-technical-design.md                  # Technical design document (evolves)
│   ├── task-edge-cases.md                        # Edge cases and decisions
│   └── diagrams/
│       ├── 01-class-diagram.puml
│       ├── 02-package-diagram.puml
│       ├── 03-sequence-{flow}.puml
│       └── 04-task-flow-dependencies.puml
├── implement/                                    # implement-flow output
│   └── flow-plan/
│       ├── task-flow-user-session-store.md       # Dynamically named task flows
│       ├── task-flow-login-form-component.md
│       ├── task-flow-auth-service.md
│       ├── task-flow-api-integration.md
│       └── task-flow-tests.md
├── test/                                         # test-flow output
├── pr/                                           # pr-flow output
│   └── feedback/
│       ├── feedback-1-review.md                  # PR feedback → knowledge record
│       └── feedback-2-review.md
├── deploy/                                       # deploy-flow output
└── docs/                                         # docs-flow output
    └── lessons-learned.md                        # Post-merge summary
```

### Files

#### task-design.md

**Purpose:** Complete design specification for the task.

**Status lifecycle:**
- `Draft` — Initial creation
- `In Review` — Developer reviewing
- `v1.0 (ENHANCED)` — After design-flow enhancement
- `v2.0 (SIGNED OFF)` — After developer approval (IMMUTABLE)

**Sections (example):**
1. Overview
2. Goals and Non-Goals
3. User Stories
4. Rules and Behavior
5. System Interactions
6. Architecture
7. Visual Specifications
8. Analytics
9. Edge Cases
10. Open Questions

**Lock after sign-off:**
```markdown
> **Status:** v2.0 (SIGNED OFF)
> **Signed Off By:** Developer Name
> **Date:** 2026-04-23
> **Immutable:** Yes
```

#### task-technical-design.md

**Purpose:** Technical design document — implementation details.

**Status lifecycle:**
- `Draft` — Created by design-flow
- `Updated` — Modified during implement-flow (deviations recorded)
- `Final` — After implementation complete

**Sections (example):**
1. Architecture Overview
2. Model Layer
3. Controller Layer
4. View Layer
5. Integration Points
6. Data Flow
7. Testing Strategy
8. Performance Considerations

**Deviations section:**
```markdown
## Deviations from task-design.md

During implementation, these deviations were necessary:

1. **task-design.md Section 4.2** specified X, but implemented Y because Z
2. **New pattern discovered:** Added caching layer not in original design
```

#### task-edge-cases.md

**Purpose:** Document edge cases, boundary conditions, and error scenarios.

**Format:**
```markdown
# Edge Cases: {Task Name}

## EC1: Empty State
**Scenario:** No items to process
**Expected:** Graceful handling, clear user message
**Implementation:** Check count before processing

## EC2: Maximum Load
**Scenario:** System under maximum load
**Expected:** Degrade gracefully, don't crash
**Implementation:** Rate limiting, queue overflow handling
```

#### Task Flows (Knowledge Records)

**Purpose:** Track implementation work and capture knowledge.

**Frontmatter:**
```yaml
---
task-flow: user-session-store
task: "user-authentication"
status: complete          # pending | in_progress | complete
depends-on: []
accepted-date: 2026-04-23
tags: ["backend", "data-model", "session"]
---
```

**Sections:**
1. Acceptance Criteria
2. Files to Create
3. Implementation Notes (added during implement-flow)
4. Patterns Used
5. Deviations from Design
6. Tests Written

**Transformation:** When the task flow is complete, it becomes a knowledge record.

#### Feedback Task Flows

**Purpose:** Capture PR feedback as structured knowledge.

**Naming:** `feedback-{N}-{reviewer}.md`

**Sections:**
1. Original Comment
2. Category (architecture, style, logic, performance)
3. Severity (blocking, suggestion, nitpick)
4. Fix Applied
5. Lesson Learned

#### Lessons Learned

**Purpose:** Post-merge summary for future reference.

**Generated by:** PR flow, `capture` mode

**Sections:**
1. What Went Well
2. What Was Missed
3. New Patterns Discovered
4. Process Improvements
5. Metrics (time spent, iterations, task flows found)

---

## Tier 2: Project-Level Knowledge Base

Shared across all tasks in the project. Updated by flows as patterns are discovered.

### Directory Structure

```
flow-storage/project/
├── ARCHITECTURE.md              # Overall architecture
├── CONVENTIONS.md               # Coding standards
├── PATTERNS.md                  # Reusable patterns catalog
└── DECISIONS.md                 # Architecture Decision Records (ADRs)
```

### Files

#### ARCHITECTURE.md

**Purpose:** High-level architecture overview.

**Created during:** ai-flow-anything initialization

**Updated by:** design flow; PR flow, `capture` mode

**Sections:**
1. Architecture Overview
2. Layer Diagram
3. Technology Stack
4. Data Flow
5. Integration Points
6. Security Model

#### CONVENTIONS.md

**Purpose:** Coding standards and conventions.

**Created during:** ai-flow-anything initialization (from profile rules + developer input)

**Updated by:** PR flow, `capture` mode (when conventions evolve)

**Sections:**
1. Naming Conventions
2. Code Style
3. Testing Standards
4. Documentation Standards
5. Git Conventions

#### PATTERNS.md

**Purpose:** Catalog of reusable patterns discovered during development.

**Created during:** ai-flow-anything initialization (empty template)

**Updated by:** implement flow; PR flow, `capture` mode

**Format:**
```markdown
# Patterns Catalog

## Pattern: Circuit Breaker
**Used by:** PaymentService, ExternalApiClient
**Context:** Preventing cascading failures in distributed systems
**Solution:** [description with code example]
**References:** PaymentService.{{ext}}, ApiClient.{{ext}}

## Pattern: Repository with Cache
**Used by:** UserRepository, ProductRepository
**Context:** Data access with caching layer
**Solution:** [description with code example]
**References:** UserRepository.{{ext}}, CacheManager.{{ext}}

## Pattern: Event Aggregation
**Used by:** OrderService, InventoryService
**Context:** Decoupling event producers and consumers
**Solution:** [description with code example]
**References:** EventAggregator.{{ext}}, OrderService.{{ext}}
```

#### DECISIONS.md

**Purpose:** Architecture Decision Records (ADRs).

**Created during:** design-flow (for each major decision)

**Format:**
```markdown
# ADR-001: Use Repository Pattern for Data Access

**Status:** Accepted
**Date:** 2026-04-23
**Context:** [why this decision was needed]
**Decision:** [what was decided]
**Consequences:** [positive and negative]
**Alternatives Considered:** [what else was evaluated]
```

---

## Tier 3: Team-Level Knowledge Base

Shared across team members. Contains onboarding and team processes.

### Directory Structure

```
flow-storage/team/
├── onboarding.md                # New team member guide
├── workflows.md                 # How the team works
└── glossary.md                  # Domain terminology
```

### Files

#### onboarding.md

**Purpose:** Guide for new team members.

**Sections:**
1. Project Overview
2. Setup Instructions
3. Architecture Overview
4. Key Patterns
5. How to Start a Task
6. Review Process
7. Deployment Process

#### workflows.md

**Purpose:** Document team workflows and processes.

**Sections:**
1. Task Development Flow
2. Code Review Process
3. Testing Requirements
4. Deployment Process
5. Incident Response

#### glossary.md

**Purpose:** Domain terminology and definitions.

**Format:**
```markdown
# Glossary

## Service
A self-contained unit of business logic that handles a specific domain.

## Handler
A component that processes incoming requests or events.

## Entity
A data object that represents a domain concept (user, order, product).
```

---

## Knowledge Base Maintenance

### Creation Triggers

| Event | KB Tier | Action |
|-------|---------|--------|
| ai-flow-anything initialization | Project, Team | Create initial structure |
| Design flow | Task | Create task KB |
| Implement flow | Task | Update task flows with implementation notes |
| PR flow, `feedback` mode | Task | Create feedback task flows |
| PR flow, `capture` mode | All tiers | Update lessons-learned, patterns, decisions |
| Code refactoring | Project | Update patterns, architecture |

### Update Rules

1. **Task-level:** Updated continuously during task development
2. **Project-level:** Updated after each task (by capture mode)
3. **Team-level:** Updated when processes change

### Search

Knowledge base is searchable. Ask the AI to search the knowledge base for a query, for example:

> "Search the knowledge base for the repository pattern."

Searches:
- All task-design.md files
- PATTERNS.md
- DECISIONS.md
- Task flows (knowledge records)
- Lessons learned

### Backups

Knowledge base is version-controlled (in `flow-storage/`). No separate backup needed.

---

## Quality Standards

### Completeness

- [ ] Every task has a knowledge base directory
- [ ] Every task flow is transformed to a knowledge record
- [ ] Every PR feedback is captured
- [ ] Patterns catalog is updated after each task
- [ ] ADRs are created for major decisions

### Accuracy

- [ ] Knowledge records match actual implementation
- [ ] Pattern references point to actual code
- [ ] ADRs reflect actual decisions, not aspirations

### Discoverability

- [ ] Clear directory structure
- [ ] Consistent naming
- [ ] Cross-references between related docs
- [ ] Searchable content
