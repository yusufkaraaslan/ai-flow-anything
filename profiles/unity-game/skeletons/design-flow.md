# Flow: Design

> **Profile:** Unity Game  
> **Purpose:** Design Unity tasks with component architecture and visual diagrams

---

## When to Use

- New game mechanics (blockers, boosters, enemies, abilities)
- UI tasks (screens, HUD elements, menus)
- System changes (save/load, audio, networking)
- Level design tools

## Input

- Task idea (from developer)
- Unity project context

## Output

```
flow-storage/tasks/{task-name}/
├── design/
│   ├── task-design.md
│   ├── task-technical-design.md
│   ├── task-edge-cases.md
│   └── diagrams/
│       ├── 01-class-diagram.puml
│       ├── 02-component-diagram.puml
│       ├── 03-sequence-{flow}.puml
│       └── 04-task-flow-dependencies.puml
└── implement/
    └── flow-plan/
        ├── task-flow-01-{descriptive-name}.md   # Dynamically named (e.g., player-health-system)
        ├── task-flow-02-{descriptive-name}.md
        └── ...                                  # AI proposes based on task analysis
```

## Prerequisites

- Unity project context loaded
- Developer can describe the task

---

## Phases

This flow has **1 review gate** (CRITICAL) in Phase 1. Phase 2 auto-executes after acceptance — its only purpose is to lock the design and commit. See `universal/workflow-structure.md` for the canonical pattern.

### Phase 1: DESIGN *(CRITICAL gate)*

**Sub-tasks (auto-proceed):**

**1.1 UNDERSTAND**
- Read `flow-storage/project/ARCHITECTURE.md`, `CONVENTIONS.md`, `PATTERNS.md`
- Explore codebase: similar tasks, reference implementations, component structure
- Check existing tasks in `flow-storage/tasks/`

**1.2 DESIGN (Visual-First)**
- Ask developer: "Describe this task in 1-2 sentences"
- Generate class diagram: MonoBehaviour components, ScriptableObject data, pure C# models/services, relationships
- Generate component diagram: GameObjects in scene hierarchy, component composition, prefab structure
- Render every `.puml` to `.svg`

**1.3 SPECIFY** — Create docs with **Unity specifics**:
- `task-design.md`: task overview, user stories, component architecture (with diagram refs), data flow, Unity integration (scenes, prefabs, ScriptableObjects), input handling, visual/audio specs, edge cases (high-level)
- `task-technical-design.md`: script breakdown (MonoBehaviour, ScriptableObject, pure C#), component responsibilities, lifecycle (Awake/Start/OnEnable/OnDestroy), data persistence, testing strategy (EditMode/PlayMode)
- `task-edge-cases.md`: scene loading/unloading, object pooling, null reference risks, performance concerns
- Sequence diagrams for complex flows (rendered)

**1.4 PLAN** — Decompose into implementation task flows (kebab-case names describing the WORK, not architectural layers):
  - Examples: `player-health-system.md`, `enemy-ai-behavior.md`, `inventory-ui-panel.md`, `save-load-system.md`, `vfx-particle-controller.md`
  - Sample plan presentation:
    ```
    Proposed task flows for "Shield Power-Up":

    1. shield-data-model.md          # ScriptableObject + stats
    2. shield-controller.md          # Activation/deactivation logic
    3. shield-visual-effect.md       # Particle system + animations
    4. shield-integration.md         # Scene setup, prefab configuration

    Suggested: Add tests.md? [Y]/[N]
    ```
  - Create task flow files with frontmatter:
    ```yaml
    ---
    task-flow: shield-data-model
    task: "shield-power-up"
    status: pending
    depends-on: []
    accepted-date: null
    tags: ["unity", "scriptable-object", "data"]
    ---
    ```
  - Define acceptance criteria per task flow
  - Build dependency graph + generate `diagrams/04-task-flow-dependencies.puml` (rendered)
  - Run internal PLAN validation (surface failures at the gate)
  - Stage for preview + compose commit message

→ **Gate: Design + plan review — [A]ccept / [F]eedback / [R]eject**  
⚠️ **This is the ONLY gate.** Accepting locks task-design.md immutable per Rule 9 AND triggers Phase 2 (auto-executes FINALIZE + COMMIT). `--no-commit` skips Phase 2's commit.

---

### Phase 2: LOCK & COMMIT

**Purpose:** Mechanically apply the decisions already reviewed and accepted at Phase 1. No separate gate.

**Sub-tasks (auto-execute after Phase 1 [A]ccept):**

**2.1 FINALIZE** — Sign off task-design.md (immutable per Rule 9).

**2.2 COMMIT** *(skipped with --no-commit)* — Stage `flow-storage/tasks/{task-name}/`; commit `design({task-name}): sign off task-design.md and {N} task flows`.

---

## Unity-Specific Design Patterns

### MonoBehaviour Components

```csharp
// Component with lifecycle
public class PlayerController : MonoBehaviour
{
    [SerializeField] private float _moveSpeed = 5f;
    [SerializeField] private PlayerModel _model;
    
    private InputProvider _input;
    
    private void Awake()
    {
        _input = new UnityInputProvider();
    }
    
    private void OnEnable()
    {
        _model.OnHealthChanged += HandleHealthChanged;
    }
    
    private void OnDisable()
    {
        _model.OnHealthChanged -= HandleHealthChanged;
    }
    
    private void Update()
    {
        var movement = _input.GetMovement();
        transform.position += movement * _moveSpeed * Time.deltaTime;
    }
    
    private void HandleHealthChanged(int newHealth)
    {
        // Update UI or visual feedback
    }
}
```

### ScriptableObject Data

```csharp
// Data asset
[CreateAssetMenu(fileName = "EnemyData", menuName = "Game/Enemy Data")]
public class EnemyData : ScriptableObject
{
    public string EnemyName;
    public int BaseHealth;
    public float MoveSpeed;
    public GameObject Prefab;
}
```

### Pure C# Model

```csharp
// No Unity dependencies
public class PlayerModel
{
    public event Action<int> OnHealthChanged;
    
    public int Health { get; private set; }
    public bool IsAlive => Health > 0;
    
    public void TakeDamage(int damage)
    {
        Health = Math.Max(0, Health - damage);
        OnHealthChanged?.Invoke(Health);
    }
}
```

---

## Customization

Users can customize:
- Add Unity-specific phases (e.g., "Prefab Setup")
- Change component templates
- Add platform-specific considerations (mobile, console, VR)