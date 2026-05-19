# Flow: Free Flow

> **Profile:** Web Frontend

## When to Use

Quick bug fixes, small enhancements, design tweaks, or minor refactors — work too small to warrant the full design-implement-review chain.

## Phases

This profile follows the canonical **3-phase, 3-gate** structure from `profiles/generic/skeletons/free-flow.md`. Web-specific guidance below.

### Phase 1: TALK *(STANDARD gate)*
- **1.1 ASK / 1.2 CONTEXT / 1.3 SCOPE** — see canonical.
- → **Gate 1: Scope review.**

### Phase 2: PLAN *(STANDARD gate)*
- **2.1 ANALYZE / 2.2 CROSS-REF / 2.3 PROPOSE** — see canonical.
- **2.1 ANALYZE** — identify impacted route definitions, component trees, and state stores.
- → **Gate 2: Plan review.**

### Phase 3: FIX *(CRITICAL gate)*
- **3.1 IMPLEMENT** — see canonical.
- **3.2 VALIDATE** — Run `npm test` (or `pnpm test` / `yarn test` per project). Web-specific checks:
  - [ ] TypeScript compilation clean (`tsc --noEmit` or `npm run typecheck`)
  - [ ] Lint clean (`npm run lint`)
  - [ ] No console.log left in production paths
  - [ ] Accessibility not regressed (color contrast, keyboard nav, screen reader)
  - If failures, fix and re-validate.
- **3.3 RECORD / 3.4 COMMIT** — see canonical.
- → **Gate 3: Fix + KB review.**

## Sub-Agent Mode & Resume Logic

See canonical: `profiles/generic/skeletons/free-flow.md`.
