# Flow: Orchestrate

> **Profile:** Mobile App

## When to Use
After design-flow when the task has 3+ task flows with independent subsets and you want parallel implementation across them.

## Phases

This profile follows the canonical **4-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/orchestrate-flow.md`). Mobile-app notes below.

### Phase 1: PLAN *(STANDARD gate)*
- **1.1 LOAD / 1.2 GROUP / 1.3 SCHEDULE** — see canonical.
- **1.2 GROUP** — when validating "no two task flows in the same wave share files," watch for these mobile shared files:
  - Dependency manifests: `pubspec.yaml`/`pubspec.lock` (Flutter), `package.json`/`yarn.lock` (RN), `Podfile`/`Podfile.lock` (iOS native), `build.gradle`/`gradle.lockfile` (Android native)
  - Route maps: `routes.dart`, `App.tsx` navigation tree, `AppRoutes.kt`
  - Native config: `Info.plist`, `AndroidManifest.xml`, `ios/Runner.xcodeproj/project.pbxproj`, `android/app/build.gradle`
  - State containers: provider/riverpod/redux store entry points
  - Localization files: `intl_*.arb`, `Localizable.strings`, `strings.xml`

→ **Gate 1: Execution plan review.**

### Phase 2: EXECUTE *(no gate)*
- **2.1 PREPARE WORKTREES** — `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {current-branch}`. Native build caches (Pods, Gradle) are per-worktree — see "build cache cost" below.
- **2.2 LAUNCH SUBAGENTS** — implement-flow in sub-agent mode (per Rule 16).
- **2.3 MONITOR / 2.4 WAIT FOR WAVE** — see canonical.

### Phase 3: MERGE *(CRITICAL gate)*
- **3.1 COLLECT / 3.2 CHERRY-PICK / 3.4 UPDATE RECORDS / 3.5 GENERATE REPORT** — see canonical.
- **3.3 VALIDATE MERGED** (mobile, substitute the project's actual stack):
  ```bash
  flutter test                      # Flutter unit + widget tests
  npm test                          # React Native (Jest)
  ```
  Native UI tests (Detox, XCTest, Espresso) run sequentially after MERGE acceptance — emulator/simulator contention makes them unsuitable for parallel orchestration.

→ **Gate 2: Final implementation review.**

### Phase 4: COMMIT *(no gate)*
- See canonical. Cleanup loop:
  ```
  for wt in .ai-workflow/worktrees/{task-name}/*/; do
    git worktree remove "$wt"
  done
  ```

## Mobile-app worktree caveats

- **Native build cache cost.** Each worktree gets its own `ios/Pods/`, `android/.gradle/`, `build/` — first build per worktree is the cold-cache slow path. On Flutter, `flutter pub get` per worktree is fast; native pod install / gradle sync is the expensive part. Plan worktree count accordingly on disk-constrained machines.
- **Lockfile churn.** Adding any dependency forces a lockfile update. Multiple parallel task flows adding deps will conflict on the lockfile at MERGE time. Resolution is typically: regenerate the lockfile from the merged manifest after cherry-pick.
- **Native code generation.** Flutter's `build_runner`, RN's codegen, and ObjC/Java bridges produce generated files that may end up tracked. If two subagents trigger codegen, the generated outputs can race — exclude generated files from cherry-pick consideration and regenerate once at MERGE if needed.
- **Emulator/simulator contention.** Don't try to run parallel UI tests across worktrees against a single emulator. UI tests stay sequential; only unit and widget tests parallelize cleanly.
