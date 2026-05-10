# Flow: Implement

> **Profile:** Mobile App

## When to Use
After design-flow.

## Phases

This profile follows the canonical **2-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/implement-flow.md`). Mobile-app specifics below.

### Phase 1: BUILD *(STANDARD gate)*
- **1.1 READ** — Load context, task flow file, design docs, completed prior task flows.
- **1.2 PLAN** — Implementation: models, screens/widgets, state management, API integration.
- **1.3 IMPLEMENT** — Create mobile code following project conventions:
  ```dart
  // Flutter: Task implementation
  class FeatureViewModel extends ChangeNotifier {
    List<Item> _items = [];
    bool _isLoading = false;

    List<Item> get items => _items;
    bool get isLoading => _isLoading;

    Future<void> loadItems() async {
      _isLoading = true;
      notifyListeners();

      _items = await _repository.fetchItems();
      _isLoading = false;
      notifyListeners();
    }
  }
  ```
- **1.4 AUTO-VALIDATE** — Run flutter test / xctest / espresso. Mobile-specific checks:
  - [ ] Platform-specific code isolated
  - [ ] Accessibility labels
  - [ ] Responsive design
  - [ ] Error handling
  - If failures, surface at the gate; do not auto-proceed.
- **1.5 DOC-SYNC** — Update task-technical-design.md with deviations; update PATTERNS.md if new patterns emerged.
- → **Gate 1: Implementation review** — files changed + mobile validation results + doc updates.

### Phase 2: COMMIT *(CRITICAL gate)*
- **2.1 UPDATE TASK FLOW** — Transform task flow file to knowledge record.
- **2.2 COMMIT** *(executes only after the gate is accepted)* — Stage only this task flow's files; commit per conventional commits. Skip 2.2 if `--no-commit`.
- → **Gate 2: Commit review** — presented BEFORE 2.2 executes git commit.

## Sub-Agent Mode

When invoked by orchestrate-flow, this flow runs in sub-agent mode (no `[A]/[F]/[R]` gates, auto-proceed, works in `.ai-workflow/worktrees/{task-name}/{task-flow}/`, returns a structured report). See canonical: `profiles/generic/skeletons/implement-flow.md` § Sub-Agent Mode and `universal/rules.md` Rule 16. No mobile-specific overrides — the same protocol applies.

## Resume Logic
Check task flow status.
