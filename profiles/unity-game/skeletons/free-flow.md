# Flow: Free Flow

> **Profile:** Unity Game

## When to Use

Quick bug fixes, small enhancements, design tweaks, or minor refactors — work too small to warrant the full design-implement-review chain.

## Phases

This profile follows the canonical **3-phase, 3-gate** structure from `profiles/generic/skeletons/free-flow.md`. Unity-specific guidance below.

### Phase 1: TALK *(STANDARD gate)*
- **1.1 ASK / 1.2 CONTEXT / 1.3 SCOPE** — see canonical.
- **1.2 CONTEXT** — when searching the KB for related work, also check scene hierarchies and prefab references for the affected GameObjects.
- → **Gate 1: Scope review.**

### Phase 2: PLAN *(STANDARD gate)*
- **2.1 ANALYZE / 2.2 CROSS-REF / 2.3 PROPOSE** — see canonical.
- **2.1 ANALYZE** — identify impacted MonoBehaviours, ScriptableObjects, prefabs, scenes, and asset references.
- → **Gate 2: Plan review.**

### Phase 3: FIX *(CRITICAL gate)*
- **3.1 IMPLEMENT** — see canonical. Follow C# patterns from the project's existing `Assets/Scripts/`.
- **3.2 VALIDATE** — Run Unity Test Runner or project test suite. Unity-specific checks:
  - [ ] No missing MonoBehaviour references in prefabs/scenes
  - [ ] No console errors on Play Mode entry
  - [ ] Serialized fields not broken by rename
  - [ ] Build succeeds (if build-relevant change)
  - If failures, fix and re-validate.
- **3.3 RECORD / 3.4 COMMIT** — see canonical.
- → **Gate 3: Fix + KB review.**

## Sub-Agent Mode & Resume Logic

See canonical: `profiles/generic/skeletons/free-flow.md`.
