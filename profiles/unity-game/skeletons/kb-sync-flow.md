# Flow: KB Sync

> **Profile:** Unity Game

## When to Use
A flow flagged a KB contradiction (Rule 18), `--status` reported drift, or work happened outside the flows.

## Phases

This profile uses the canonical **2-phase, 2-gate** kb-sync-flow from `profiles/generic/skeletons/kb-sync-flow.md` (Phase 1: SYNC → Phase 2: COMMIT). Unity-specific reality signals for sub-task 1.2 below.

## Unity Reality Signals (sub-task 1.2 REALITY SCAN)

| KB claim | Reality source |
|----------|----------------|
| Unity version | `ProjectSettings/ProjectVersion.txt` |
| Installed packages / test framework | `Packages/manifest.json` (`com.unity.test-framework`, etc.) |
| Assembly / namespace structure claims | `*.asmdef` files and actual `Assets/Scripts/` layout |
| Render pipeline (Built-in / URP / HDRP) | `Packages/manifest.json` + `ProjectSettings/GraphicsSettings.asset` |
| Scene list / build claims | `ProjectSettings/EditorBuildSettings.asset` |
| Patterns in PATTERNS.md (ScriptableObject configs, event channels, etc.) | Referenced `.cs` files still exist and still implement the pattern |
| Input system claims (legacy vs new Input System) | `Packages/manifest.json` (`com.unity.inputsystem`) + `ProjectSettings/InputManager.asset` |

## Customization
Add project-specific signals (e.g. "addressable groups match documented content packs").
