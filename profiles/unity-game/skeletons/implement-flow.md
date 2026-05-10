# Flow: Implement

> **Profile:** Unity Game  
> **Purpose:** Implement Unity tasks with proper lifecycle and component architecture

---

## When to Use

After the design flow completes and design is signed off.

## Input

- Task flow files from `flow-storage/tasks/{task-name}/implement/flow-plan/`
- task-design.md and task-technical-design.md
- Unity project context

## Output

- C# scripts (MonoBehaviour, ScriptableObject, pure C#)
- Unity assets (prefabs, ScriptableObject assets)
- Tests (EditMode/PlayMode)
- Updated task flow files (knowledge records)

## Prerequisites

- Design signed off
- Task flow files created

---

## Phases

This profile follows the canonical **2-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/implement-flow.md`). Unity-specific guidance below.

### Phase 1: BUILD *(STANDARD gate)*

**Sub-tasks (auto-proceed):**

**1.1 READ** — Load project context, task flow file (acceptance criteria, files to create), design docs (relevant sections), completed prior task flows, reference implementations in codebase.

**1.2 PLAN** — Map acceptance criteria to **Unity patterns**:
- MonoBehaviour lifecycle
- Component composition
- ScriptableObject usage
- Assets needed (scripts, prefabs, ScriptableObject assets)
- Tests (EditMode for logic, PlayMode for integration)

**1.3 IMPLEMENT** — Write Unity code following project conventions:
```csharp
// Model (pure C#)
public class FeatureModel { ... }

// Controller (MonoBehaviour)
public class FeatureController : MonoBehaviour { ... }

// View (MonoBehaviour)
public class FeatureView : MonoBehaviour { ... }

// Data (ScriptableObject)
[CreateAssetMenu(...)]
public class FeatureData : ScriptableObject { ... }
```
- Implement lifecycle: `Awake()` (internal init) → `OnEnable()` (subscribe) → `Start()` (external init) → `OnDisable()` (unsubscribe) → `OnDestroy()` (cleanup)
- Use `[SerializeField]` for inspector visibility, `RequireComponent` where appropriate, `Header`/`Tooltip` attributes
- Follow profile rules: no `UnityEngine` in pure C# layers, Views use `Init()` not `[Inject]`, proper event cleanup

**1.4 AUTO-VALIDATE** — Run Unity tests (EditMode + PlayMode). Unity-specific checks:
- [ ] No `using UnityEngine;` in Model/Domain files
- [ ] Views use `[SerializeField]` not public fields
- [ ] Lifecycle methods implemented
- [ ] Events unsubscribed in OnDisable/OnDestroy
- [ ] No GameObject.Find in Update
- [ ] Object pooling considered
- [ ] Naming conventions followed
- If failures, surface at the gate; do not auto-proceed.

**1.5 DOC-SYNC** — Update task-technical-design.md with deviations; update task-edge-cases.md with discoveries; update PATTERNS.md if new pattern found.

→ **Gate 1: Implementation review** — files changed + Unity validation results + doc updates + component hierarchy (if applicable).

---

### Phase 2: COMMIT *(CRITICAL gate)*

**Sub-tasks (auto-proceed):**

**2.1 UPDATE TASK FLOW** — Transform task flow file to knowledge record:
```yaml
status: complete
accepted-date: {date}
```
Append: scripts created, prefabs created, components used, deviations from design.

**2.2 COMMIT** *(executes only after the gate is accepted)* — Stage scripts and assets; commit per conventional commits:
```
feat({task-name}): {task-flow-name}

- Add FeatureModel, FeatureController, FeatureView
- Add FeatureData ScriptableObject
- Implement lifecycle and event handling

Refs: {task-flow-name}
```
Skip 2.2 if `--no-commit`.

→ **Gate 2: Commit review** — presented BEFORE 2.2 executes git commit.

---

## Unity Implementation Patterns

### Component with Events

```csharp
public class HealthComponent : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    
    public event Action<int, int> OnHealthChanged; // current, max
    public event Action OnDied;
    
    public int CurrentHealth { get; private set; }
    
    private void Awake()
    {
        CurrentHealth = _maxHealth;
    }
    
    public void TakeDamage(int damage)
    {
        if (CurrentHealth <= 0) return;
        
        CurrentHealth = Mathf.Max(0, CurrentHealth - damage);
        OnHealthChanged?.Invoke(CurrentHealth, _maxHealth);
        
        if (CurrentHealth == 0)
            OnDied?.Invoke();
    }
    
    public void Heal(int amount)
    {
        if (CurrentHealth <= 0) return;
        
        CurrentHealth = Mathf.Min(_maxHealth, CurrentHealth + amount);
        OnHealthChanged?.Invoke(CurrentHealth, _maxHealth);
    }
}
```

### ScriptableObject Configuration

```csharp
[CreateAssetMenu(fileName = "WeaponConfig", menuName = "Game/Weapon Config")]
public class WeaponConfig : ScriptableObject
{
    public string WeaponName;
    public int Damage;
    public float FireRate;
    public float Range;
    public GameObject ProjectilePrefab;
    public AudioClip FireSound;
}
```

### Object Pooling

```csharp
public class ObjectPool : MonoBehaviour
{
    [SerializeField] private GameObject _prefab;
    [SerializeField] private int _initialSize = 10;
    
    private Queue<GameObject> _pool = new Queue<GameObject>();
    
    private void Awake()
    {
        for (int i = 0; i < _initialSize; i++)
            CreateNewObject();
    }
    
    private void CreateNewObject()
    {
        var obj = Instantiate(_prefab, transform);
        obj.SetActive(false);
        _pool.Enqueue(obj);
    }
    
    public GameObject Get()
    {
        if (_pool.Count == 0)
            CreateNewObject();
        
        var obj = _pool.Dequeue();
        obj.SetActive(true);
        return obj;
    }
    
    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        _pool.Enqueue(obj);
    }
}
```

---

## Resume Logic

Check for existing progress:
1. Read all task flow files
2. Find first pending/in_progress
3. Ask: "Resume from {task-flow-name}?"

---

## Customization

Users can customize:
- Add Unity-specific validations
- Change component templates
- Add platform-specific code (mobile input, VR, etc.)

---

## Sub-Agent Mode

When this flow is invoked as a **sub-agent** by the orchestrate-flow, it runs in a different mode:

**Trigger:** The orchestrator passes `mode: sub-agent` in its launch prompt.

**Behavior changes:**
- **No `[A]/[F]/[R]` gates.** All sub-tasks auto-proceed without user-facing interruption. The orchestrator handles the only gate.
- **Works in an isolated git worktree** at `.ai-workflow/worktrees/{task-name}/{task-flow}/`.
- **Commits in the worktree** at the end of Phase 2. Do not push.
- **Returns a structured implementation report** instead of presenting gate artifacts. Format:

```markdown
## Sub-Agent Report: {task-flow} ({task-name})

**Status:** success | failure
**Worktree:** .ai-workflow/worktrees/{task-name}/{task-flow}/
**Commit SHA:** <sha>

### Files Created
- path/to/file1
- path/to/file2

### Files Modified
- path/to/file3

### Test Results
- {passing}/{total} passing (EditMode: X, PlayMode: Y)
- Coverage delta: +X%

### Deviations from Design
- {deviation 1}
- {deviation 2}

### Issues Encountered
- {issue 1}
- {issue 2}
```
