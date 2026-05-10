# Flow: Implement

> **Profile:** Backend API

## When to Use
After design-flow.

## Phases

This profile follows the canonical **2-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/implement-flow.md`). Backend-API specifics below.

### Phase 1: BUILD *(STANDARD gate)*
- **1.1 READ** — Load context, task flow file, design docs, completed prior task flows.
- **1.2 PLAN** — Implementation: models, services, endpoints, tests.
- **1.3 IMPLEMENT** — Create backend code following project conventions:
  ```python
  # Model
  class User(BaseModel):
      id: int
      email: EmailStr
      name: str
      created_at: datetime

  # Repository
  class UserRepository:
      def __init__(self, db: Session):
          self.db = db

      def find_by_id(self, user_id: int) -> User | None:
          return self.db.query(User).filter(User.id == user_id).first()

  # Service
  class UserService:
      def __init__(self, repo: UserRepository):
          self.repo = repo

      def get_user(self, user_id: int) -> User:
          user = self.repo.find_by_id(user_id)
          if not user:
              raise UserNotFoundError(user_id)
          return user

  # Endpoint
  @app.get("/users/{user_id}")
  async def get_user(
      user_id: int,
      service: UserService = Depends()
  ):
      return service.get_user(user_id)
  ```
- **1.4 AUTO-VALIDATE** — Run pytest. Backend-specific checks:
  - [ ] Type hints
  - [ ] Input validation
  - [ ] Error handling
  - [ ] HTTP status codes
  - If failures, surface at the gate; do not auto-proceed.
- **1.5 DOC-SYNC** — Update task-technical-design.md with deviations; update PATTERNS.md if new patterns emerged.
- → **Gate 1: Implementation review** — files changed + backend validation results + doc updates.

### Phase 2: COMMIT *(CRITICAL gate)*
- **2.1 UPDATE TASK FLOW** — Transform task flow file to knowledge record.
- **2.2 COMMIT** *(executes only after the gate is accepted)* — Stage only this task flow's files; commit per conventional commits. Skip 2.2 if `--no-commit`.
- → **Gate 2: Commit review** — presented BEFORE 2.2 executes git commit.

## Sub-Agent Mode

When invoked by orchestrate-flow, this flow runs in sub-agent mode (no `[A]/[F]/[R]` gates, auto-proceed, works in `.ai-workflow/worktrees/{task-name}/{task-flow}/`, returns a structured report). See canonical: `profiles/generic/skeletons/implement-flow.md` § Sub-Agent Mode and `universal/rules.md` Rule 16. No backend-API-specific overrides — the same protocol applies.

## Resume Logic
Check task flow status.
