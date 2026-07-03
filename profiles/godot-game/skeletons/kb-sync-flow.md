# Flow: KB Sync

> **Profile:** Godot Game

## When to Use
A flow flagged a KB contradiction (Rule 18), `--status` reported drift, or work happened outside the flows.

## Phases

This profile uses the canonical **2-phase, 2-gate** kb-sync-flow from `profiles/generic/skeletons/kb-sync-flow.md` (Phase 1: SYNC → Phase 2: COMMIT). Godot-specific reality signals for sub-task 1.2 below.

## Godot Reality Signals (sub-task 1.2 REALITY SCAN)

Check every KB claim in these categories against the live project:

| KB claim | Reality source |
|----------|----------------|
| Autoload list / count ("exactly N autoloads") | `[autoload]` section of `project.godot` |
| Test framework (GUT vs gdUnit4 vs custom) | `addons/gut/` vs `addons/gdUnit4/` presence; `extends GutTest` vs `extends GdUnitTestSuite` in `tests/` |
| Documented test commands | Do the paths in the command exist (`addons/gdUnit4/bin/GdUnitCmdTool.gd`)? Does the Godot version match `project.godot` `config/features`? |
| Godot version / renderer / physics engine | `project.godot` (`config/features`, `rendering/`, `physics/`) |
| Scene / script structure claims | Actual `scenes/`, `scripts/`, `src/` layout |
| Signal architecture patterns in PATTERNS.md | Referenced `.gd` files still exist and still declare those signals |
| Input actions documented | `[input]` section of `project.godot` |
| Export targets / release claims | `export_presets.cfg` |

## Customization
Add project-specific signals (e.g. "theme count matches `themes/*.tres`").
