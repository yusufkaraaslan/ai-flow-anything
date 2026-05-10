# Flow: Design

> **Profile:** Mobile App

## When to Use
New screens, tasks, or system changes.

## Output
```
flow-storage/tasks/{task-name}/
├── design/
│   ├── task-design.md
│   ├── task-technical-design.md
│   ├── task-edge-cases.md
│   └── diagrams/
└── implement/
    └── flow-plan/
        ├── task-flow-01-{descriptive-name}.md   # Dynamically named (e.g., login-screen)
        ├── task-flow-02-{descriptive-name}.md
        └── ...                                  # AI proposes based on task analysis
```

## Phases

This flow has **1 review gate** (CRITICAL) in Phase 1. Phase 2 auto-executes after acceptance — its only purpose is to lock the design and commit. See `universal/workflow-structure.md` for the canonical pattern.

### Phase 1: DESIGN *(CRITICAL gate)*

**Sub-tasks (auto-proceed):**

**1.1 UNDERSTAND** — Load context, explore codebase (screens/, widgets/, services/, navigation/).

**1.2 DESIGN (Visual-First)** — Generate screen flow + widget/component diagrams (class + package + sequence as needed). Render every `.puml` to `.svg`.

**1.3 SPECIFY** — Create docs with **mobile-app specifics**:
  - Screen architecture
  - State management
  - Navigation flow
  - Platform-specific code
  - Accessibility
  - Performance

**1.4 PLAN** — Decompose into implementation task flows (kebab-case names):
  - Examples: `login-screen.md`, `user-profile-widget.md`, `push-notification-service.md`
  - Create files with frontmatter + tags
  - Generate task-flow dependency diagram
  - Suggested: offer to add `tests.md` as optional task flow
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

## Mobile Patterns

```dart
// Flutter: Screen with state management
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Consumer<HomeViewModel>(
        builder: (context, viewModel, child) {
          if (viewModel.isLoading)
            return Center(child: CircularProgressIndicator());
          return ListView.builder(
            itemCount: viewModel.items.length,
            itemBuilder: (context, index) => 
              ListTile(title: Text(viewModel.items[index].title)),
          );
        },
      ),
    );
  }
}
```