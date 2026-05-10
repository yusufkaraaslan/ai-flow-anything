# Flow: Orchestrate

> **Profile:** Unity Game

## When to Use
After design-flow when the task has 3+ task flows with independent subsets and you want parallel implementation across them.

⚠️ **Unity-specific limitation:** Unity Hub does not support opening multiple Editor instances on the same project. Subagents in this profile must use **CLI-only test execution** (no Editor PlayMode tests). If the task requires PlayMode integration tests, fall back to sequential implement-flow instead.

## Phases

This profile follows the canonical **4-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/orchestrate-flow.md`). Unity-specific notes below.

### Phase 1: PLAN *(STANDARD gate)*
- **1.1 LOAD / 1.2 GROUP / 1.3 SCHEDULE** — see canonical.
- **1.2 GROUP** — when validating "no two task flows in the same wave share files," watch for these Unity-specific shared files:
  - `*.meta` files (every asset has a sibling .meta with a GUID — concurrent asset additions race here)
  - `*.unity` scene files (especially scenes referenced by multiple subagents)
  - `*.prefab` prefab assets
  - `ProjectSettings/*.asset` (TagManager, InputManager, GraphicsSettings, etc.)
  - `Packages/manifest.json`, `Packages/packages-lock.json`

→ **Gate 1: Execution plan review.**

### Phase 2: EXECUTE *(no gate)*
- **2.1 PREPARE WORKTREES** — `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {current-branch}`. Each worktree gets its own `Library/` cache on first compile (large; ensure disk space).
- **2.2 LAUNCH SUBAGENTS** — implement-flow in sub-agent mode (per Rule 16).
- **2.3 MONITOR / 2.4 WAIT FOR WAVE** — see canonical.

### Phase 3: MERGE *(CRITICAL gate)*
- **3.1 COLLECT / 3.2 CHERRY-PICK / 3.4 UPDATE RECORDS / 3.5 GENERATE REPORT** — see canonical.
- **3.3 VALIDATE MERGED** (Unity-specific, CLI):
  ```bash
  Unity -batchmode -nographics -projectPath . \
    -runTests -testPlatform EditMode \
    -testResults TestResults.xml -quit
  ```
  PlayMode tests are skipped during orchestration — run them in a final sequential pass after MERGE acceptance.

→ **Gate 2: Final implementation review.**

### Phase 4: COMMIT *(no gate)*
- See canonical. Cleanup loop:
  ```
  for wt in .ai-workflow/worktrees/{task-name}/*/; do
    git worktree remove "$wt"
  done
  ```

## Unity-specific worktree caveats

- **`.meta` file GUIDs.** Two parallel subagents creating new assets generate independent GUIDs in the `.meta` files. Cherry-picking both is fine — the GUIDs do not collide. But if both subagents touch the *same* existing `.meta` (e.g. moving an asset), expect a conflict.
- **`Library/` cache** is per-worktree, regenerated on first compile. First subagent in a fresh worktree pays the cold-cache cost.
- **Editor cannot be open** while Unity CLI compiles in worktrees. If the developer has the Editor open on the main worktree, it will lose script-reload state when MERGE lands changes.
