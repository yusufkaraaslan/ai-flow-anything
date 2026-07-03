# Discovery Guide: Unity Game

> **Purpose:** Instructions for AI to explore Unity projects and discover architecture

---

## Step 1: Project Identity

### Slot: project-name

**Discovery:**
1. Read `ProductName` from `ProjectSettings/ProjectSettings.asset`
2. If not found, read first line of `README.md`
3. If still unclear, ask developer

### Slot: game-module-name

**Discovery:**
1. Scan `Assets/` for directories that:
   - Are NOT standard Unity folders (`Plugins`, `Editor`, `Resources`, `StreamingAssets`, `TextMesh Pro`)
   - Contain `Scripts/` subdirectory
   - Have `Controller/`, `Model/`, or `View/` subdirectories (if MVC)
2. List candidates and ask developer to confirm

**Example:**
```
Found potential game modules:
  1. Assets/GameCore/ (contains Scripts/)
  2. Assets/Gameplay/ (contains Scripts/)

Which is the primary game module?
```

---

## Step 2: Framework Stack

### Slot: di-framework

**Discovery:**
Search code for dependency injection patterns:
- Zenject: `[Inject]`, `Container.Bind`, `MonoInstaller`
- VContainer: `[Inject]`, `IContainerBuilder`
- Reflex: `[Inject]`, `Container`
- Extenject: `Zenject` namespace

If found, record:
- Framework name
- Version (from `Packages/manifest.json`)
- Main installer file location

### Slot: async-library

**Discovery:**
Search for async patterns:
- `UniTask` (most common in Unity)
- `async/await` with `Task`
- Coroutines (`IEnumerator`, `StartCoroutine`)
- `async void` (fire-and-forget)

Record:
- Primary async pattern
- Cancellation token usage
- Common patterns (e.g., `CancellationTokenSource` fields)

### Slot: animation-library

**Discovery:**
Search for animation patterns:
- DOTween: `DOSequence`, `DOMove`, `DOTween.Kill`
- LeanTween: `LeanTween.move`, `LeanTween.alpha`
- PrimeTween: `Tween.Position`, `Tween.Delay`
- Unity Animation: `Animator`, `Animation`
- Custom tweening

---

## Step 3: Architecture

### Slot: architecture-pattern

**Discovery:**
1. Check directory structure under `Assets/{module}/Scripts/`:
   - `Model/`, `Controller/`, `View/` → MVC
   - `Domain/`, `Application/`, `Infrastructure/` → Clean/Hexagonal
   - `Components/`, `Systems/` → ECS (or component-based)
   - Flat structure → Component-based (Unity default)

2. Search for base classes:
   - Abstract `*Model`, `*Controller`, `*View` → MVC
   - `MonoBehaviour` subclasses everywhere → Component-based
   - `IComponentData`, `ISystem` → DOTS/ECS

3. Ask developer if ambiguous

### Slot: base-classes

**Discovery:**
For each layer (Model, Controller, View, etc.):
1. Find abstract classes
2. Find interfaces (`I*Model`, `I*Controller`)
3. Record:
   - Class/interface name
   - File path
   - Key methods/properties

### Slot: feature-categories

**Discovery:**
Scan `Scripts/` subdirectories:
- `Enemies/`, `Abilities/`, `Items/` → Game mechanics
- `UI/`, `HUD/`, `Menus/` → Interface
- `Audio/`, `VFX/` → Presentation
- `AI/`, `Pathfinding/` → Systems

Ask developer to name and categorize.

---

## Step 4: Integration Points

### Slot: signal-system

**Discovery:**
Search for inter-system communication:
- Zenject `SignalBus`
- Custom event bus
- C# events (`event Action`)
- UnityEvents
- Observer pattern

Record:
- Mechanism name
- Signal/event names
- Publishers and subscribers

### Slot: level-system

**Discovery:**
Search for level/scene management:
- `SceneManager`
- Custom level loader
- Addressables
- Asset bundles
- JSON/XML level data

### Slot: data-persistence

**Discovery:**
Search for save/load:
- `PlayerPrefs`
- JSON serialization
- Binary serialization
- SQLite
- Cloud save

---

## Step 5: Testing

### Slot: test-framework

**Discovery:**
Check for:
- Unity Test Framework (built-in)
- `Tests/` directories
- Test attributes (`[Test]`, `[UnityTest]`)
- Third-party frameworks (NUnit, xUnit)

### Slot: test-assembly

**Discovery:**
Find test assembly definitions:
- `*.asmdef` files in `Tests/`
- EditMode vs PlayMode tests

---

## Step 6: Documentation

### Slot: docs-directory

**Discovery:**
Check for:
- `Docs/` or `docs/`
- `Documentation~/`
- `README.md` in `Assets/`

**If not found:** Suggest creating `Docs/` at project root

---

## Discovery Summary

After discovery, present:

```
Discovery Complete: Unity Game Profile

Project: {project-name}
  Module: {game-module}
  Namespace: {root-namespace}

Framework:
  DI: {di-framework}
  Async: {async-library}
  Animation: {animation-library}

Architecture:
  Pattern: {architecture-pattern}
  Base Classes: {list}
  Features: {categories}

Integration:
  Signals: {signal-system}
  Levels: {level-system}
  Save: {data-persistence}

Testing:
  Framework: {test-framework}
  Assembly: {test-assembly}

Questions:
  1. {if anything unclear}
```
