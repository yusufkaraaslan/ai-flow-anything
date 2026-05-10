# Flow: Implement

> **Profile:** Web Frontend

## When to Use
After design-flow.

## Phases

This profile follows the canonical **2-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/implement-flow.md`). Web-frontend specifics below.

### Phase 1: BUILD *(STANDARD gate)*
- **1.1 READ** — Load context, task flow file, design docs, completed prior task flows.
- **1.2 PLAN** — Component implementation: component structure, state management, API calls, styling, accessibility approach.
- **1.3 IMPLEMENT** — Create frontend code following project conventions:
  ```tsx
  export const FeatureComponent: React.FC = () => {
    const [data, setData] = useState<Data>();

    useEffect(() => {
      fetchData().then(setData);
    }, []);

    return (
      <div className="feature">
        {data ? <DataView data={data} /> : <Loading />}
      </div>
    );
  };
  ```
- **1.4 AUTO-VALIDATE** — Run jest/vitest. Frontend-specific checks:
  - [ ] No `any` types
  - [ ] Accessibility attributes
  - [ ] Responsive design
  - [ ] Error handling
  - If failures, surface at the gate; do not auto-proceed.
- **1.5 DOC-SYNC** — Update task-technical-design.md with deviations; update PATTERNS.md if new patterns emerged.
- → **Gate 1: Implementation review** — files changed + frontend validation results + doc updates.

### Phase 2: COMMIT *(CRITICAL gate)*
- **2.1 UPDATE TASK FLOW** — Transform task flow file to knowledge record.
- **2.2 COMMIT** *(executes only after the gate is accepted)* — Stage only this task flow's files; commit per conventional commits. Skip 2.2 if `--no-commit`.
- → **Gate 2: Commit review** — presented BEFORE 2.2 executes git commit.

## Sub-Agent Mode

When invoked by orchestrate-flow, this flow runs in sub-agent mode (no `[A]/[F]/[R]` gates, auto-proceed, works in `.ai-workflow/worktrees/{task-name}/{task-flow}/`, returns a structured report). See canonical: `profiles/generic/skeletons/implement-flow.md` § Sub-Agent Mode and `universal/rules.md` Rule 16. No web-frontend-specific overrides — the same protocol applies.

## Resume Logic
Check task flow status.
