# Flow: KB Sync

> **Profile:** Mobile App

## When to Use
A flow flagged a KB contradiction (Rule 18), `--status` reported drift, or work happened outside the flows.

## Phases

This profile uses the canonical **2-phase, 2-gate** kb-sync-flow from `profiles/generic/skeletons/kb-sync-flow.md` (Phase 1: SYNC → Phase 2: COMMIT). Mobile-specific reality signals for sub-task 1.2 below.

## Mobile Reality Signals (sub-task 1.2 REALITY SCAN)

| KB claim | Reality source |
|----------|----------------|
| Framework + version (Flutter / React Native) | `pubspec.yaml` / `package.json` + lockfiles |
| Test framework + documented test commands | Dev dependencies (`flutter_test`, `jest`, `detox`); do the commands reference real paths? |
| State management claims (Riverpod/Bloc/Redux/...) | Dependency list + actual imports in `lib/` or `src/` |
| Navigation claims | Router dependency + route definitions in code |
| Platform config claims (min SDK, permissions, bundle ids) | `android/app/build.gradle`, `ios/Runner/Info.plist`, `app.json` |
| Screen/module structure claims | Actual `lib/` / `src/` layout |
| Patterns in PATTERNS.md | Referenced files still exist and still implement the pattern |

## Customization
Add project-specific signals (e.g. "feature flags in KB match remote-config keys").
