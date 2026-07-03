# Flow: KB Sync

> **Profile:** Backend API

## When to Use
A flow flagged a KB contradiction (Rule 18), `--status` reported drift, or work happened outside the flows.

## Phases

This profile uses the canonical **2-phase, 2-gate** kb-sync-flow from `profiles/generic/skeletons/kb-sync-flow.md` (Phase 1: SYNC → Phase 2: COMMIT). Backend-specific reality signals for sub-task 1.2 below.

## Backend Reality Signals (sub-task 1.2 REALITY SCAN)

| KB claim | Reality source |
|----------|----------------|
| Framework + version (FastAPI/Django/Express/Go/...) | `pyproject.toml` / `package.json` / `go.mod` / `Cargo.toml` |
| Test framework + documented test commands | Dev dependencies + config (`pytest.ini`, `pyproject.toml [tool.pytest]`); do the commands run against real paths? |
| Service list / architecture claims | `docker-compose.yml` services, actual `src/`/`app/` module layout |
| Database + migration claims | Migration directory (`migrations/`, `alembic/`), ORM config |
| API route/endpoint structure claims | Actual router/controller files |
| Environment/config claims | `.env.example`, settings modules, deployment config |
| Patterns in PATTERNS.md (repository, service layer, middleware) | Referenced files still exist and still implement the pattern |

## Customization
Add project-specific signals (e.g. "queue topology in KB matches broker config").
