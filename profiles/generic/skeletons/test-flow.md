# Flow: Test

> **Profile:** Generic  
> **Purpose:** Plan, generate, and validate tests for tasks

---

## When to Use

- After the implement flow completes (for new tasks)
- Standalone for existing code that lacks tests
- Before PR to ensure coverage

## Input

- Task docs (task-design.md, task-technical-design.md)
- Implementation code
- Edge cases from task-edge-cases.md

## Output

- Test files
- Coverage report
- Updated task flow files

## Prerequisites

- Test framework is identified (from discovery)
- Task implementation exists (or is planned)

---

## Phases

This flow has **2 phases, 2 gates**. Each phase contains multiple sub-tasks that auto-proceed without intermediate gates. test-flow does not git commit — tests are committed via implement-flow or pr-flow alongside the code they cover. See `universal/workflow-structure.md`.

---

### Phase 1: WRITE

**Purpose:** Read context, plan the test suite, generate stubs, and fill them in as one reviewable package.

**Sub-tasks (auto-proceed):**

**1.1 READ**
- Read task docs: `task-design.md` (requirements), `task-technical-design.md` (technical details), `task-edge-cases.md` (boundary conditions)
- Read implementation code
- Identify testable units: functions/methods, classes/modules, integration points
- Check existing tests: what's already covered, what's missing

**1.2 PLAN**
- Define test categories: unit (business logic), integration (component interaction), edge case (boundaries, errors), E2E (if applicable)
- Map each acceptance criterion → test case(s); each edge case → test case
- Identify mocks/stubs and test data needed

**1.3 GENERATE**
- Create test files following project conventions:
  ```
  {{test-dir}}/
  ├── {{task-name}}_test.{{ext}}
  └── ... (number and names vary by project)
  ```
- Write test stubs: function signatures, Arrange-Act-Assert structure, comments describing intent
- Import dependencies, set up fixtures/mocks

**1.4 IMPLEMENT**
- Fill in unit tests (happy path, error, boundary)
- Fill in integration tests (component interaction, data flow)
- Fill in edge case tests (from `task-edge-cases.md` and discoveries)
- Use descriptive names: `test_calculate_total_with_discount()`, `test_reject_negative_quantity()`

**Artifacts (presented at gate):**
- Test plan outline (categories + mapping to acceptance criteria)
- Test files with implementations
- Mock/fixture setup

**Gate:** Test code review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

---

### Phase 2: VERIFY

**Purpose:** Run the suite, capture results, and document findings.

**Sub-tasks (auto-proceed):**

**2.1 VALIDATE**
- Run tests using the project's framework (NEVER skip tests — user global rule):
  ```bash
  pytest tests/test_{task-name}_*.py    # Python
  npm test -- test/{task-name}          # JavaScript
  go test ./...                         # Go
  ```
- Capture pass/fail per test + coverage delta vs baseline
- If failures, do **not** auto-proceed — surface at the Phase 2 gate so developer can [F]eedback to fix code or test

**2.2 DOCUMENT**
- Append a test summary to `flow-storage/tasks/{task-name}/test/`
- Update `task-technical-design.md`: testing approach used, coverage achieved, known gaps
- Update task flow file: mark tests complete, add notes
- Update `flow-storage/project/PATTERNS.md` if new testing patterns emerged

**Artifacts (presented at gate):**
- Test results (pass/fail breakdown)
- Coverage report + delta
- Updated docs (TDD, task flow, PATTERNS.md)

**Gate:** Results review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

Note: test-flow does not have a COMMIT phase. Tests are committed by implement-flow (alongside the code) or pr-flow (as part of validation). **After accepting this gate, the generated test files exist on disk but are NOT committed.** Run implement-flow or pr-flow next to land them in version control.

---

## Test Naming Conventions

**Good:**
```python
def test_user_can_login_with_valid_credentials():
def test_login_rejects_empty_password():
def test_login_locks_after_three_failed_attempts():
```

**Bad:**
```python
def test1():
def test_login():
def test_stuff():
```

---

## Customization

Users can customize:
- Add test categories (performance, security)
- Change coverage threshold
- Add test data generators
- Add mocking frameworks
