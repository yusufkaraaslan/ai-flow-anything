# Flow: Docs

> **Profile:** Generic  
> **Purpose:** Maintain and update project documentation

---

## When to Use

- Documentation is outdated
- New team member joining
- After major architecture changes
- Periodic documentation review

## Input

- Existing docs in `docs/`
- Source code
- Knowledge base records

## Output

- Updated documentation
- New documentation
- Removed outdated docs

---

## Phases

This flow has **2 phases, 2 gates**. Each phase contains multiple sub-tasks that auto-proceed without intermediate gates. See `universal/workflow-structure.md` for the canonical pattern.

---

### Phase 1: APPLY

**Purpose:** Audit existing docs, propose changes, and apply them as one reviewable package.

**Sub-tasks (auto-proceed):**

**1.1 SCAN (audit)**
- List all documentation:
  - `flow-storage/project/*.md`
  - `flow-storage/tasks/*/*.md`
  - `flow-storage/team/*.md`
  - `README.md`, `CHANGELOG.md`, any other `.md` files
- Check each for:
  - Outdated content (references old versions)
  - Missing documentation (new tasks without docs)
  - Broken links
  - Placeholder text (`{placeholder}`, `TODO`, `FIXME`)
- Compare docs to codebase:
  - Tasks documented vs tasks implemented
  - APIs documented vs APIs existing
- Build an audit report (rationale per finding)

**1.2 PROPOSE**
- For each finding, draft the change as a diff (or new file outline)
- Cross-check against living-document policy (Rule 2): update in place, no archives, no v2 copies
- Group diffs by target file

**1.3 APPLY**
- For each accepted diff, modify the doc:
  - Read current version
  - Apply update (content, diagrams, links, code examples)
  - Verify no broken cross-references introduced
- Create missing task docs:
  ```
  flow-storage/tasks/{task-name}/
  ├── design/task-design.md
  ├── design/task-technical-design.md
  └── README.md
  ```
- Add API documentation, ADRs, navigation links as needed
- Re-run completeness checks: all tasks documented, all APIs documented, all ADRs present, code examples work, diagrams match code, links work

**Artifacts (presented at gate):**
- Audit summary (counts: outdated / missing / broken)
- Diff list (file → change summary)
- Modified files
- New files

**Gate:** Doc updates review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

---

### Phase 2: COMMIT

**Purpose:** Commit all doc changes as one atomic unit.

**Sub-tasks (auto-proceed):**

**2.1 COMMIT** *(executes only after Phase 2 gate is accepted)*
- Stage updated `flow-storage/` and `docs/` files (and any `AGENTS.md` / `CLAUDE.md` edits if those were part of scope)
- Compose commit message:
  ```
  docs: update {scope} — {one-line summary}

  Files updated: {list}
  ```
- Run `git commit`

**Artifacts (presented at gate):**
- Staged file list (`git diff --cached --stat`)
- Proposed commit message

**Gate:** Commit review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE C — CRITICAL)*

**Failure modes to verify before [A]ccept:**
- Files staged: only docs touched by this run — no code edits sneaking in (docs-flow never touches code)
- Living-document compliance: did the diffs update files in place, or did they create `*-v2.md` / archive copies? The latter violates Rule 2
- `task-design.md` untouched (Rule 9 — immutable)

**Opt-out:** `--no-commit` skips sub-task 2.1.

---

## Documentation Standards

### File Organization

```
flow-storage/
├── project/
│   ├── ARCHITECTURE.md
│   ├── CONVENTIONS.md
│   ├── PATTERNS.md
│   └── DECISIONS.md
├── tasks/
│   └── {task-name}/
│       ├── design/
│       │   ├── task-design.md
│       │   ├── task-technical-design.md
│       │   ├── task-edge-cases.md
│       │   └── diagrams/
│       ├── implement/
│       │   └── flow-plan/
│       ├── test/
│       ├── pr/
│       │   └── feedback/
│       ├── deploy/
│       └── docs/
│           └── lessons-learned.md
└── team/
    ├── onboarding.md
    ├── workflows.md
    └── glossary.md
```

### Naming

- Use `lowercase-with-dashes.md`
- `README.md` for directory overviews
- Be descriptive: `authentication-flow.md` not `auth.md`

### Structure

```markdown
# Title

> **Status:** Draft | Current | Outdated  
> **Last Updated:** YYYY-MM-DD

## Overview
Brief description.

## Details
...

## Examples
```code
...
```

## See Also
- [Related Doc](../related.md)
```

---

## Customization

Users can customize:
- Add documentation categories
- Change verification checklist
- Add style guide enforcement
- Add diagram generation
- Add auto-linking
