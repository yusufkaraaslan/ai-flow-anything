# Universal Rules

> **Applies to:** ALL projects, ALL profiles, ALL flows  
> **Purpose:** Non-negotiable guardrails that never change regardless of tech stack

---

## Rule 1: Documentation Before Code

No implementation may begin without an approved design document.

**Enforcement:**
- The design flow MUST run before the implement flow
- The implement flow checks for `task-design.md` before starting
- If `task-design.md` is missing, the implement flow exits with: "Run the design flow for `{name}` first"

**Exception:** Hotfixes and one-line changes may skip design phase, but must be documented retroactively.

---

## Rule 2: Living Documents

All documentation in `docs/` is living — updated at every workflow step, never archived.

**Practices:**
- task-design.md is locked after sign-off (immutable), but deviations are recorded in task-technical-design.md
- task-technical-design.md evolves during implementation
- task-edge-cases.md grows as new edge cases are discovered
- Task flows transform into knowledge records after completion
- Feedback task flows become permanent knowledge records

**Anti-pattern:** Creating `docs-v2/`, `docs-archive/`, or dated copies. Update in place.

---

## Rule 3: Unlimited Iteration Review Gates

Every phase ends with developer review. Options are always:

- **[A]ccept** — Proceed to next phase
- **[F]eedback** — Provide feedback, flow revises and re-presents
- **[R]eject** — Abandon this approach, start over

**Constraints:**
- No maximum iteration count
- Never auto-accept even if "perfect"
- Developer must explicitly type "A" or "Accept"
- Feedback must be specific and actionable

---

## Rule 4: Diagrams Are Mandatory

Every task design must produce at minimum:

1. **Class diagram** — Core abstractions and relationships
2. **Package/module diagram** — Dependencies and layering
3. **Sequence diagram** — For any non-trivial flow (optional for trivial flows)

**Standards:**
- Follow `universal/diagram-standards.md`
- Default format: PlantUML
- User can override per project
- Diagrams are reviewed alongside text documents
- Diagrams live in `flow-storage/tasks/{task-name}/design/diagrams/`
- **Source AND rendered image both required.** Every `.puml` / `.d2` / `.mmd` must have a sibling `.svg` (preferred) or `.png` in the same directory. An unrendered diagram source is not a deliverable — readers shouldn't need to install a renderer locally to see what was designed. Mermaid is exempt only when the docs are read on a Mermaid-rendering platform (GitHub, GitLab, Notion). See `universal/diagram-standards.md` "Rendering" for the install commands.

---

## Rule 5: Knowledge Capture

After every task, the following must be updated:

**Per-task:**
- `lessons-learned.md` — What went well, what was missed
- `feedback/` — PR feedback records (if any)

**Project-level:**
- `flow-storage/project/PATTERNS.md` — New reusable patterns discovered
- `flow-storage/project/DECISIONS.md` — Architecture decisions (ADRs)

**Team-level:**
- `flow-storage/team/workflows.md` — Process improvements

---

## Rule 6: No Auto-Approval

The AI never decides a phase is "good enough" without developer input.

**Anti-patterns:**
- "This looks good, proceeding..."
- "Implementation complete, moving to PR..."
- Auto-accepting after N iterations

**Correct:**
- "Phase complete. Review and type [A]ccept, [F]eedback, or [R]eject"
- Waiting for explicit developer input before proceeding

---

## Rule 7: Atomic Commits

Each task flow (from implement-flow) becomes one commit.

**Format:**
```
feat({task-name}): {task-flow-name} — brief description

- Detailed change 1
- Detailed change 2

Refs: {task-flow-name}
```

**Example:**
```
feat(user-authentication): implement login validation

- Add UserValidator with email format checking
- Implement password strength requirements
- Add session token generation

Refs: #124
```

---

## Rule 8: Task Flow Frontmatter Schema

All generated task flow files must use this frontmatter:

```yaml
---
task-flow: {task-flow-name}
task: "{task-name}"
status: pending          # pending | in_progress | complete
depends-on: [{dependencies}]
accepted-date: null       # set when developer accepts implementation
---
```

**Fields:**
`task-flow` — Short identifier (kebab-case, e.g., user-session, auth-service, login-form)
- `task-name` — Full task name (kebab-case or PascalCase)
- `status` — Current state
- `depends-on` — List of task flow names this depends on
- `accepted-date` — Date of developer acceptance (ISO 8601)

---

## Rule 9: Design Lock After Sign-Off

After developer accepts task-design.md, it becomes immutable:

```markdown
> **Status:** v2.0 (ENHANCED) — Signed Off
> **Signed Off By:** {developer name}
> **Date:** {date}
> **Immutable:** Yes
```

**Implications:**
- Changes after lock require new version or new task
- Deviations during implementation are recorded in task-technical-design.md, not task-design.md
- Post-merge, edge cases and deviations feed into next iteration

---

## Rule 10: Context Loading First

Every flow must load project context before doing anything else.

**Context sources (in order):**
1. `flow-storage/project/ARCHITECTURE.md` — Overall architecture
2. `flow-storage/project/CONVENTIONS.md` — Coding standards
3. `flow-storage/project/PATTERNS.md` — Reusable patterns
4. Prior task docs in `flow-storage/tasks/` — Reference implementations
5. `.ai-workflow/rules.md` — Project-specific overrides

**Loading pattern:**
```markdown
## First Step: Load Context

Before doing anything else, read:
1. `flow-storage/project/ARCHITECTURE.md`
2. `flow-storage/project/CONVENTIONS.md`
3. `flow-storage/project/PATTERNS.md`

This provides: architecture rules, framework patterns, coding standards, 
and reference implementations. **Do not skip this step.**
```

---

## Rule 11: Transparent Execution

The AI must show what it's doing, not just do it.

**Required transparency:**
- List files being read
- Show key decisions made
- Display progress through phases
- Explain why a pattern was chosen
- Cite reference implementations

**Example:**
```
Phase 1: READ complete

Task flow: controller.md
  Requirements: 4 acceptance criteria extracted
  Files to create: 5 identified

Design docs loaded:
  task-design.md Section 4.3 (Controller Layer)
  task-technical-design.md Controller section
  task-edge-cases.md: 3 relevant edge cases

Reference implementation:
  OrderController.{{ext}} (API controller pattern)
  UserService.{{ext}} (service layer pattern)

Proceeding to Phase 2: PLAN...
```

---

## Rule 12: No Placeholders in Final Output

After generation, verify no `{placeholder}` text remains in non-template files.

**Check:** Search for `\{[a-z][a-z0-9_-]*\}` pattern in generated files.
**Exception:** Template files meant for developer customization (e.g., `DESIGN-TEMPLATE.md`)

---

## Rule 13: Test Coverage for New Tasks

Every task must have tests.

**Minimum:**
- Unit tests for business logic
- Integration tests for system boundaries
- Edge case tests from task-edge-cases.md

**Coverage target:** 80% for new code (profile may override)

---

## Rule 14: Backward Compatibility

Changes must not break existing features.

**Checks:**
- Run existing tests before and after changes
- Verify no regressions in related features
- Update integration points, don't break them

---

## Rule 15: Accessibility and Inclusivity

(For UI projects) All user-facing features must consider accessibility.

**Checks:**
- Color contrast
- Keyboard navigation
- Screen reader compatibility
- Touch target sizes

---

## Rule 16: Sub-Agent Gate Suppression

When a flow is invoked as a sub-agent by an orchestrator, internal `[A]/[F]/[R]` gates are suppressed. The sub-agent auto-proceeds through all sub-tasks without user-facing gates and returns a structured report to the orchestrator.

**Enforcement:**
- The orchestrator passes a `mode: sub-agent` flag in its launch prompt.
- The sub-agent detects this flag and disables all gate prompts.
- Gate artifacts (validation results, implementation notes, deviated designs) are wrapped into the sub-agent's return report instead of being presented to the user.
- The orchestrator evaluates each sub-agent's report and decides whether to surface issues at its own gate.

**Sub-agent return report format:**
```markdown
## Sub-Agent Report: {task-flow-name} ({task-name})

**Status:** success | failure
**Worktree:** .ai-workflow/worktrees/{task-name}/{task-flow}/
**Commit SHA:** <sha>

### Files Created
- path/to/file1
- path/to/file2

### Files Modified
- path/to/file3

### Test Results
- {passing}/{total} passing
- Coverage delta: +X%

### Deviations from Design
- {deviation 1}
- {deviation 2}

### Issues Encountered
- {issue 1}
- {issue 2}
```

---

## Rule 17: Git Worktree Isolation

Parallel task flows in the orchestrate-flow use isolated git worktrees. Each sub-agent works in its own worktree to avoid cross-contamination.

**Worktree conventions:**
- Base path: `.ai-workflow/worktrees/{task-name}/{task-flow}/`
- Created via: `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {base-branch}`
- Each worktree is an independent checkout of the same branch — no sharing files between concurrent subagents.
- Subagents commit in their worktree as normal (conventional commits per Rule 7).
- The orchestrator cherry-picks worktree commits into the main branch during Phase 3 MERGE.
- Worktrees are cleaned up (`git worktree remove`) after the final consolidation commit.

**Guardrails (two-layer enforcement):**
- **Design-time check** — design-flow Phase 1.4 PLAN validates pairwise file overlap: any two task flows with no `depends-on` between them but overlapping declared file lists must either gain a `depends-on` (forcing serial) or be merged. Verifiable from declared file lists; no `parallel-with` field is required.
- **Orchestrate-time check** — orchestrate-flow Phase 1.2 GROUP re-validates at wave granularity after computing dependency waves: no two task flows in the same wave may share files. This catches anything the design-time check missed (e.g. files added to the lists after design sign-off).
- **Last-resort detection** — if both layers missed a conflict, the orchestrator detects it as a cherry-pick merge conflict in Phase 3.2 and surfaces it to the developer at the MERGE gate.
- **No pushing from worktrees** — subagents commit locally; pushes happen (if at all) from the main worktree after MERGE acceptance.

---

## Enforcement

These rules are enforced by:
1. **Flows** — Check rules at each phase gate
2. **Validators** — Run automated checks where possible
3. **Developer review** — Human verifies at [A]/[F]/[R] gates
4. **Knowledge base** — Violations recorded in lessons-learned

**Severity levels:**
- **error** — Must fix before proceeding (Rule 1, 2, 4, 6, 8, 9, 16)
- **warning** — Should fix, can proceed with justification (Rule 3, 7, 10, 11, 12, 17)
- **info** — Best practice, noted but not blocking (Rule 5, 13, 14, 15)
