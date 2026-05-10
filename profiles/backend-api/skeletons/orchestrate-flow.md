# Flow: Orchestrate

> **Profile:** Backend API

## When to Use
After design-flow when the task has 3+ task flows with independent subsets (e.g. multiple endpoints, separate services, isolated migrations) and you want parallel implementation.

## Phases

This profile follows the canonical **4-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/orchestrate-flow.md`). Backend-API notes below.

### Phase 1: PLAN *(STANDARD gate)*
- **1.1 LOAD / 1.2 GROUP / 1.3 SCHEDULE** — see canonical.
- **1.2 GROUP** — when validating "no two task flows in the same wave share files," watch for these backend shared files:
  - Dependency manifests: `requirements.txt`, `pyproject.toml`, `go.mod`/`go.sum`, `Cargo.toml`/`Cargo.lock`, `package.json`/lockfile
  - Route registration: `urls.py`, `app.py`/`main.py` (FastAPI/Flask), `router.go`, `Startup.cs`
  - Migration files: most ORMs serialize migration filenames by timestamp — two parallel task flows generating migrations at the same second collide
  - Settings/config: `settings.py`, `config.go`, `appsettings.json`
  - DI registrations: `containers.py`, service container setup

→ **Gate 1: Execution plan review.**

### Phase 2: EXECUTE *(no gate)*
- **2.1 PREPARE WORKTREES** — `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {current-branch}`. Each worktree needs its own virtual environment / dependency cache — see "isolation strategy" below.
- **2.2 LAUNCH SUBAGENTS** — implement-flow in sub-agent mode (per Rule 16).
- **2.3 MONITOR / 2.4 WAIT FOR WAVE** — see canonical.

### Phase 3: MERGE *(CRITICAL gate)*
- **3.1 COLLECT / 3.2 CHERRY-PICK / 3.4 UPDATE RECORDS / 3.5 GENERATE REPORT** — see canonical.
- **3.3 VALIDATE MERGED** (backend-API; substitute the project's actual stack):
  ```bash
  pytest                       # Python
  go test ./...                # Go
  npm test                     # Node
  cargo test                   # Rust
  ```
  Plus migration replay against a clean test database to catch ordering issues from parallel migration generation.

→ **Gate 2: Final implementation review.**

### Phase 4: COMMIT *(no gate)*
- See canonical. Cleanup loop:
  ```
  for wt in .ai-workflow/worktrees/{task-name}/*/; do
    git worktree remove "$wt"
  done
  ```

## Backend-API worktree caveats

- **Database state.** Parallel subagents running tests against a shared dev database race each other. Strongly prefer per-worktree test databases (separate file for SQLite; separate schema for Postgres) or use the framework's transactional test rollback. Detail in `flow-storage/project/CONVENTIONS.md` if your project has a standard pattern.
- **Migration filename collisions.** Two parallel task flows generating migrations within the same second produce identical or near-identical filenames. The orchestrator surfaces these as cherry-pick conflicts at MERGE; resolution is usually rename-and-renumber.
- **Virtual env / dependency isolation.** Per-worktree `venv/` (Python), `vendor/` (Go), or `node_modules/` (Node) keep dependency state isolated. First install in each worktree is the slow path.
- **External service contention.** If subagents hit the same external service (e.g. a shared dev queue, a single Stripe test account), serialize those tests instead of running them in parallel. The PLAN gate is the time to flag this.
