# Profile: Godot Game

> **Purpose:** Godot/GDScript game projects  
> **When to Use:** Godot 3.x or 4.x projects

---

## Description

Optimized for Godot game development with GDScript or C#.

Supports:
- GDScript (primary)
- C# (via Mono/.NET)
- Any Godot version (3.x, 4.x)

---

## Detection Hints

**Indicators:**
- `project.godot` exists
- `*.gd` files present
- `scene/` or `scenes/` directory
- `*.tscn` files present

**Confidence:**
- 3-4 matches: "This is definitely a Godot project"

---

## Capabilities

All 8 flows with Godot-specific patterns:
- Node-based architecture
- GDScript conventions
- Signal system
- Scene inheritance

## Slots Discovered

- project-name, game-module, root-namespace
- script-language (GDScript/C#)
- node-patterns, signal-system
- scene-structure

---

## Godot-Specific Conventions

- Node composition over inheritance
- Signals for communication
- Scene instancing and preloading
- `_ready()`, `_process()`, `_physics_process()` lifecycle

See `rules.md` for complete convention list.
