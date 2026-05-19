# Flow: Free Flow

> **Profile:** Godot Game

## When to Use

Quick bug fixes, small enhancements, design tweaks, or minor refactors — work too small to warrant the full design-implement-review chain.

## Phases

This profile follows the canonical **3-phase, 3-gate** structure from `profiles/generic/skeletons/free-flow.md`. Godot-specific guidance below.

### Phase 1: TALK *(STANDARD gate)*
- **1.1 ASK / 1.2 CONTEXT / 1.3 SCOPE** — see canonical.
- **1.2 CONTEXT** — when searching the KB for related work, also scan scene files (`.tscn`) for node references near the affected area.
- → **Gate 1: Scope review.**

### Phase 2: PLAN *(STANDARD gate)*
- **2.1 ANALYZE / 2.2 CROSS-REF / 2.3 PROPOSE** — see canonical.
- → **Gate 2: Plan review.**

### Phase 3: FIX *(CRITICAL gate)*
- **3.1 IMPLEMENT** — see canonical. GDScript patterns:
  ```gdscript
  extends Node
  class_name MyFeature

  @export var setting: int = 0

  func _ready():
      pass
  ```
- **3.2 VALIDATE** — Run gdUnit4/GUT tests. Godot-specific checks:
  - [ ] Type hints present
  - [ ] Signals connected (no leaks)
  - [ ] Node paths valid
  - [ ] No `print()` left in (use `push_warning()` or logging)
  - If failures, fix and re-validate.
- **3.3 RECORD / 3.4 COMMIT** — see canonical.
- → **Gate 3: Fix + KB review.**

## Sub-Agent Mode & Resume Logic

See canonical: `profiles/generic/skeletons/free-flow.md`.
