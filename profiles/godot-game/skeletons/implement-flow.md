# Flow: Implement

> **Profile:** Godot Game

## When to Use
After design-flow completes.

## Phases

This profile follows the canonical **2-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/implement-flow.md`). Godot-specific guidance below.

### Phase 1: BUILD *(STANDARD gate)*
- **1.1 READ** — Load context, task flow file, design docs, completed prior task flows.
- **1.2 PLAN** — GDScript implementation: script structure, node hierarchy, signal connections.
- **1.3 IMPLEMENT** — Create GDScript files following project conventions:
  ```gdscript
  extends Node
  class_name FeatureController

  @export var model: FeatureModel
  @onready var view = $FeatureView

  func _ready():
      model.updated.connect(_on_model_updated)

  func _on_model_updated():
      view.update_display(model.data)
  ```
- **1.4 AUTO-VALIDATE** — Run gdUnit4/GUT tests. Godot-specific checks:
  - [ ] Type hints used
  - [ ] Signals connected/disconnected (no leaks)
  - [ ] Node paths valid
  - [ ] `_ready()` initialization correct
  - If failures, surface at the gate; do not auto-proceed.
- **1.5 DOC-SYNC** — Update task-technical-design.md with deviations; update PATTERNS.md if new patterns emerged.
- → **Gate 1: Implementation review** — files changed + Godot validation results + doc updates.

### Phase 2: COMMIT *(CRITICAL gate)*
- **2.1 UPDATE TASK FLOW** — Transform task flow file to knowledge record (status: complete, accepted-date, notes).
- **2.2 COMMIT** *(executes only after the gate is accepted)* — Stage only this task flow's files; commit per conventional commits. Skip 2.2 if `--no-commit`.
- → **Gate 2: Commit review** — presented BEFORE 2.2 executes git commit.

## Sub-Agent Mode

When invoked by orchestrate-flow, this flow runs in sub-agent mode (no `[A]/[F]/[R]` gates, auto-proceed, works in `.ai-workflow/worktrees/{task-name}/{task-flow}/`, returns a structured report). See canonical: `profiles/generic/skeletons/implement-flow.md` § Sub-Agent Mode and `universal/rules.md` Rule 16. No Godot-specific overrides — the same protocol applies.

## Resume Logic
Check task flow status and resume.
