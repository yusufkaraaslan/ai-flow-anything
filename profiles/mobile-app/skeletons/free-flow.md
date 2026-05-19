# Flow: Free Flow

> **Profile:** Mobile App

## When to Use

Quick bug fixes, small enhancements, design tweaks, or minor refactors — work too small to warrant the full design-implement-review chain.

## Phases

This profile follows the canonical **3-phase, 3-gate** structure from `profiles/generic/skeletons/free-flow.md`. Mobile-specific guidance below.

### Phase 1: TALK *(STANDARD gate)*
- **1.1 ASK / 1.2 CONTEXT / 1.3 SCOPE** — see canonical.
- → **Gate 1: Scope review.**

### Phase 2: PLAN *(STANDARD gate)*
- **2.1 ANALYZE / 2.2 CROSS-REF / 2.3 PROPOSE** — see canonical.
- **2.1 ANALYZE** — identify impacted screens, navigation routes, state management stores, and platform-specific code paths.
- → **Gate 2: Plan review.**

### Phase 3: FIX *(CRITICAL gate)*
- **3.1 IMPLEMENT** — see canonical.
- **3.2 VALIDATE** — Run `flutter test` / `npm test` (per project). Mobile-specific checks:
  - [ ] Platform parity: change works on both iOS and Android (or explicitly scoped to one)
  - [ ] Navigation flow not broken
  - [ ] No layout overflow issues on small screens
  - [ ] Permission changes documented (if new permissions required)
  - If failures, fix and re-validate.
- **3.3 RECORD / 3.4 COMMIT** — see canonical.
- → **Gate 3: Fix + KB review.**

## Sub-Agent Mode & Resume Logic

See canonical: `profiles/generic/skeletons/free-flow.md`.
