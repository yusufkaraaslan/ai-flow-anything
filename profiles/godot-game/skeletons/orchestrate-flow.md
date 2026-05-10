# Flow: Orchestrate

> **Profile:** Godot Game

## When to Use
After design-flow when the task has 3+ task flows with independent subsets and you want parallel implementation across them.

## Phases

This profile follows the canonical **4-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/orchestrate-flow.md`). Godot-specific notes below.

### Phase 1: PLAN *(STANDARD gate)*
- **1.1 LOAD / 1.2 GROUP / 1.3 SCHEDULE** — see canonical.
- **1.2 GROUP** — when validating "no two task flows in the same wave share files," watch for these Godot-specific shared files:
  - `project.godot` (autoload registration, input map, project settings)
  - `*.tscn` scene files (especially main scene, level scenes)
  - `default_bus_layout.tres`, `*.tres` resource configs
  - `addons/*` plugin code

→ **Gate 1: Execution plan review.**

### Phase 2: EXECUTE *(no gate)*
- **2.1 PREPARE WORKTREES** — `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {current-branch}`.
- **2.2 LAUNCH SUBAGENTS** — implement-flow in sub-agent mode (per Rule 16). Each subagent runs the Godot implement-flow it finds in `.ai-workflow/flows/implement-flow.md`.
- **2.3 MONITOR / 2.4 WAIT FOR WAVE** — see canonical.

### Phase 3: MERGE *(CRITICAL gate)*
- **3.1 COLLECT / 3.2 CHERRY-PICK / 3.4 UPDATE RECORDS / 3.5 GENERATE REPORT** — see canonical.
- **3.3 VALIDATE MERGED** (Godot-specific):
  ```bash
  godot --headless --path . -s addons/gdUnit4/bin/GdUnitCmdTool.gd -a tests/
  ```
  (substitute `addons/gut/gut_cmdln.gd -gtest=res://tests/` if the project uses GUT instead of gdUnit4)

→ **Gate 2: Final implementation review.**

### Phase 4: COMMIT *(no gate)*
- See canonical. Cleanup loop:
  ```
  for wt in .ai-workflow/worktrees/{task-name}/*/; do
    git worktree remove "$wt"
  done
  ```

## Godot-specific worktree caveats

- **`.import/` cache is per-worktree.** Each fresh worktree triggers a full asset reimport on first `godot --headless` invocation — slow but unavoidable. Subagents should expect a one-time delay.
- **Editor instance contention.** Godot allows multiple editor instances on different paths simultaneously (unlike Unity), so worktrees can run in parallel without conflict.
- **Autoload registration** lives in `project.godot`. If two parallel task flows both register a new autoload, the cherry-pick will conflict on `project.godot`. The orchestrator surfaces this at MERGE.
