# Profile: Unity Game

> **Purpose:** Unity/C# game projects using any architecture pattern  
> **When to Use:** Unity projects with C# scripting

---

## Description

Optimized for Unity game development with C#. Supports any architecture:
- MVC (Model-View-Controller)
- MVVM (Model-View-ViewModel)
- ECS (Entity Component System)
- Component-based (Unity default)
- Custom architectures

Works with any Unity version and any frameworks:
- DI frameworks: Zenject, VContainer, Reflex, etc.
- Async: UniTask, Coroutines, async/await
- Animation: DOTween, LeanTween, PrimeTween, etc.
- Networking: Mirror, Netcode, Photon, etc.

---

## Detection Hints

**Indicators:**
- `Assets/` directory exists
- `*.cs` files present
- `ProjectSettings/ProjectSettings.asset` exists
- `Packages/manifest.json` exists
- `*.unity` scene files present

**Confidence:**
- 4-5 matches: "This is definitely a Unity project"
- 3 matches: "Likely Unity — ask developer to confirm"
- < 3 matches: "Not confident"

---

## Capabilities

| Flow | Description |
|--------|-------------|
| **design-flow** | Task design with Unity-specific diagrams and component architecture |
| **implement-flow** | Per-task-flow implementation with C# patterns and Unity lifecycle |
| **free-flow** | Quick fixes, tweaks, small refactors with Unity-specific validation (missing references, console errors) |
| **parallel-implement-flow** | Parallel implementation of all task flows via worktrees + subagents |
| **pr-flow** | PR validation with Unity-specific checks (scene references, prefab integrity) |
| **test-flow** | Unity Test Framework (EditMode/PlayMode) integration |
| **deploy-flow** | Unity build pipeline and store submission |
| **docs-flow** | Unity docs upkeep with component diagrams |
| **kb-sync-flow** | Reconcile KB + task records with the live Unity project (manifest, ProjectSettings, asmdefs) |

---

## What Gets Discovered

| Slot | Discovery Method | Fallback |
|------|-----------------|----------|
| project-name | `ProjectSettings.asset` or README | Ask developer |
| game-module | `Assets/` subdirectory with `Scripts/` | Ask developer |
| root-namespace | Most frequent namespace in code | Ask developer |
| di-framework | Search for Zenject/VContainer/Reflex | Ask developer |
| async-library | Search for UniTask/Coroutine patterns | Ask developer |
| animation-library | Search for DOTween/LeanTween | Ask developer |
| feature-categories | Scan `Scripts/` subdirectories | Ask developer |
| base-classes | Find abstract classes in Model/Controller | Ask developer |
| test-framework | Check for Unity Test Framework | Assume built-in |

---

## Unity-Specific Conventions

This profile enforces:
- Separation of UnityEngine dependencies from pure C# layers
- Proper MonoBehaviour lifecycle (Awake, Start, OnEnable, OnDisable, OnDestroy)
- Component-based architecture patterns
- Unity-specific testing (EditMode/PlayMode)
- Scene and prefab integrity

See `rules.md` for complete convention list.

---

## Example Projects

- Mobile puzzle games
- 2D/3D platformers
- RPGs and strategy games
- VR/AR experiences
- Unity-based tools and editors
