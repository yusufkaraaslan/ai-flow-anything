# Flow: KB Sync

> **Profile:** Web Frontend

## When to Use
A flow flagged a KB contradiction (Rule 18), `--status` reported drift, or work happened outside the flows.

## Phases

This profile uses the canonical **2-phase, 2-gate** kb-sync-flow from `profiles/generic/skeletons/kb-sync-flow.md` (Phase 1: SYNC → Phase 2: COMMIT). Frontend-specific reality signals for sub-task 1.2 below.

## Frontend Reality Signals (sub-task 1.2 REALITY SCAN)

| KB claim | Reality source |
|----------|----------------|
| Framework + version (React/Vue/Svelte/Angular) | `package.json` dependencies + lockfile |
| Test framework (Jest/Vitest/Playwright/Cypress) | `package.json` devDependencies + config files (`vitest.config.*`, `playwright.config.*`) |
| Documented commands (`npm run ...`) | `package.json` `scripts` section — do the named scripts exist? |
| Build tool (Vite/webpack/Next) | Config file presence + `package.json` |
| State management / routing claims | Dependencies (`zustand`, `@tanstack/react-query`, `react-router`, etc.) + actual imports in `src/` |
| Component/directory structure claims | Actual `src/` layout |
| Patterns in PATTERNS.md (hooks, HOCs, store slices) | Referenced files still exist and still implement the pattern |
| Styling approach (Tailwind/CSS modules/styled) | `tailwind.config.*`, dependency list, actual usage in components |

## Customization
Add project-specific signals (e.g. "design tokens in KB match `tokens.json`").
