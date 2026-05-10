# Flow: Design

> **Profile:** Backend API

## When to Use
New endpoints, services, or system changes.

## Output
```
flow-storage/tasks/{task-name}/
├── design/
│   ├── task-design.md
│   ├── task-technical-design.md
│   ├── task-edge-cases.md
│   └── diagrams/
└── implement/
    └── flow-plan/
        ├── task-flow-01-{descriptive-name}.md   # Dynamically named (e.g., user-repository)
        ├── task-flow-02-{descriptive-name}.md
        └── ...                                  # AI proposes based on task analysis
```

## Phases

This flow has **1 review gate** (CRITICAL) in Phase 1. Phase 2 auto-executes after acceptance — its only purpose is to lock the design and commit. See `universal/workflow-structure.md` for the canonical pattern.

### Phase 1: DESIGN *(CRITICAL gate)*

**Sub-tasks (auto-proceed):**

**1.1 UNDERSTAND** — Load context, explore codebase (controllers/, services/, models/, migrations/).

**1.2 DESIGN (Visual-First)** — Generate API flow + data model diagrams (class + package + sequence as needed). Render every `.puml` to `.svg`.

**1.3 SPECIFY** — Create docs with **backend-api specifics**:
  - API specification (OpenAPI/GraphQL schema)
  - Data models
  - Service interfaces
  - Authentication/authorization
  - Error handling
  - Database schema changes

**1.4 PLAN** — Decompose into implementation task flows (kebab-case names):
  - Examples: `user-data-model.md`, `auth-service.md`, `api-endpoint.md`, `db-migration.md`
  - Create files with frontmatter + tags
  - Generate task-flow dependency diagram
  - Suggested: offer to add `tests.md` as optional task flow
  - Run internal PLAN validation (surface failures at the gate)
  - Stage for preview + compose commit message

→ **Gate: Design + plan review — [A]ccept / [F]eedback / [R]eject**  
⚠️ **This is the ONLY gate.** Accepting locks task-design.md immutable per Rule 9 AND triggers Phase 2 (auto-executes FINALIZE + COMMIT). `--no-commit` skips Phase 2's commit.

---

### Phase 2: LOCK & COMMIT

**Purpose:** Mechanically apply the decisions already reviewed and accepted at Phase 1. No separate gate.

**Sub-tasks (auto-execute after Phase 1 [A]ccept):**

**2.1 FINALIZE** — Sign off task-design.md (immutable per Rule 9).

**2.2 COMMIT** *(skipped with --no-commit)* — Stage `flow-storage/tasks/{task-name}/`; commit `design({task-name}): sign off task-design.md and {N} task flows`.

---

## Backend Patterns

```python
# FastAPI endpoint
@app.post("/users", response_model=UserResponse)
async def create_user(
    request: CreateUserRequest,
    service: UserService = Depends()
):
    user = service.create_user(request)
    return UserResponse.from_model(user)

# Service layer
class UserService:
    def __init__(self, repo: UserRepository):
        self.repo = repo
    
    def create_user(self, request: CreateUserRequest) -> User:
        user = User(**request.dict())
        return self.repo.save(user)
```