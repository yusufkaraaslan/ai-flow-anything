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

Every STANDARD and CRITICAL phase gate presents developer review. Options are always:

- **[A]ccept** — Proceed to next phase
- **[F]eedback** — Provide feedback, flow revises and re-presents
- **[R]eject** — Abandon this approach, start over

**Constraints:**
- No maximum iteration count
- Never auto-accept even if "perfect"
- Developer must explicitly type "A" or "Accept"
- Feedback must be specific and actionable

**Scope:** Gate types and cadence are defined in `universal/workflow-structure.md` "Phase Gate Protocol". LIGHT (Gate Type A) phases are context-loading steps that produce no reviewable artifact — they report what was loaded and auto-proceed with an interrupt window. They are not review gates and this rule does not apply to them. Every phase that produces artifacts ends in a STANDARD or CRITICAL gate.

---

## Rule 4: Diagrams Are Mandatory (Scaled by Task Class)

Every task is classified at design time (design-flow sub-task 1.2) as one of two classes, recorded as `Task Class:` in the canonical status block (Rule 9):

- **system** — new systems, architecture changes, refactors, cross-cutting integrations, anything with non-trivial control flow or a data model. **Diagrams REQUIRED** (severity: error).
- **content** — assets, audio, VFX, UI theming/polish, copy, configuration, data-only additions. **Diagrams OPTIONAL** (severity: warning): may be skipped only when task-design.md records a one-line justification, e.g. "Diagrams skipped: content task — no new control flow or data model."

Default when ambiguous: **system**. The developer can override the classification at the design gate.

For **system** tasks (and content tasks that opt in), the design must produce at minimum:

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

**Clarification:** LIGHT gates (Gate Type A in `universal/workflow-structure.md`) are not auto-approval — they cover context-loading phases that produce no reviewable artifact. Every phase that produces artifacts ends in a STANDARD or CRITICAL gate; the developer decides there.

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
- `task` — Full task name (kebab-case or PascalCase)
- `status` — Current state
- `depends-on` — List of task flow names this depends on
- `accepted-date` — Date of developer acceptance (ISO 8601)
- `tags` — Optional list of tags (e.g., `["backend", "data-model"]`)

---

## Rule 9: Design Lock After Sign-Off

After developer accepts task-design.md, it becomes immutable. Sign-off state lives in the **canonical status block** — a strict, machine-greppable schema:

```markdown
> **Status:** SIGNED OFF
> **Version:** v1.0
> **Task Class:** system
> **Signed Off By:** {developer name}
> **Date:** {ISO-8601 date}
> **Immutable:** Yes
```

**Schema rules (strict):**
- **Exactly one status block per document**, located immediately after the H1 title. No secondary status markers anywhere else in the file (no "Signed Off" footers, no per-section status lines).
- `Status:` is one of exactly: `Draft` → `In Review` → `SIGNED OFF`. Nothing else.
- `Version:` starts at `v1.0`. It never changes after sign-off (post-sign-off changes are forbidden — that's the point of this rule).
- `Task Class:` is `system` or `content` (Rule 4).
- `Immutable:` is `Yes` if and only if `Status:` is `SIGNED OFF`; otherwise `No`.
- Before sign-off: `Status: Draft` or `In Review`, `Signed Off By: —`, `Immutable: No`.
- Sign-off (design-flow sub-task 2.1 FINALIZE) **updates this block in place** — it never appends a second block or a "signed off" section elsewhere.

**Machine check** (used by design-flow gates, status mode, and kb-sync-flow):
```
grep -c '^> \*\*Status:\*\*' task-design.md         # must equal 1
tail -n +10 task-design.md | grep -ci 'signed off'  # must equal 0 — no sign-off wording outside the status block
```

**Implications:**
- Changes after lock require a new task (or an explicit, developer-initiated re-open recorded in DECISIONS.md)
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

**Trust, but verify:** loaded KB content is only useful if it is true. Rule 18 requires a freshness spot-check as part of context loading.

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

Parallel task flows in the parallel-implement-flow use isolated git worktrees. Each sub-agent works in its own worktree to avoid cross-contamination.

**Worktree conventions:**
- Base path: `.ai-workflow/worktrees/{task-name}/{task-flow}/`
- Created via: `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {base-branch}`
- Each worktree is an independent checkout of the same branch — no sharing files between concurrent subagents.
- Subagents commit in their worktree as normal (conventional commits per Rule 7).
- Parallel-implement-flow cherry-picks worktree commits into the main branch during Phase 3 MERGE.
- Worktrees are cleaned up (`git worktree remove`) after the final consolidation commit.

**Guardrails (two-layer enforcement):**
- **Design-time check** — design-flow Phase 1.4 PLAN validates pairwise file overlap: any two task flows with no `depends-on` between them but overlapping declared file lists must either gain a `depends-on` (forcing serial) or be merged. Verifiable from declared file lists; no `parallel-with` field is required.
- **Wave-time check** — parallel-implement-flow Phase 1.2 GROUP re-validates at wave granularity after computing dependency waves: no two task flows in the same wave may share files. This catches anything the design-time check missed (e.g. files added to the lists after design sign-off).
- **Last-resort detection** — if both layers missed a conflict, parallel-implement-flow detects it as a cherry-pick merge conflict in Phase 3.2 and surfaces it to the developer at the MERGE gate.
- **No pushing from worktrees** — subagents commit locally; pushes happen (if at all) from the main worktree after MERGE acceptance.

---

## Rule 18: Knowledge Base Freshness

A stale knowledge base is worse than an empty one: Rule 10 injects KB content as trusted context at the top of every flow run, so wrong facts poison every downstream decision.

**Load-time spot-check (every flow):** as part of context loading (Rule 10), verify 2–3 load-bearing KB claims against observable reality before proceeding. Minimum set:
- **Test framework** — does the framework named in ARCHITECTURE.md / onboarding.md match what's actually in the project (addon directories, dev dependencies, test file imports)?
- **Count/constraint claims** — do numeric constraints (e.g. "exactly N autoloads", "M services") match the current config/manifest?
- **Phase/status labels** — does the project phase or status in the KB match the root README / CLAUDE.md / AGENTS.md?

**On contradiction:**
- Flag it at the current flow's next gate: "KB says X, project shows Y."
- Recommend running **kb-sync-flow** to repair.
- Do NOT silently trust the stale claim, and do NOT silently fix the KB mid-flow (that derails the current flow and hides the drift from the developer).

**Staleness signal:** project- and team-level KB files carry a `> **Last Synced:** {date}` header line, stamped by kb-sync-flow. Status mode reports the age; flows may warn when it exceeds the team's cadence.

**Active repair:** kb-sync-flow (see `universal/workflow-structure.md`) is the flow that walks every KB entry and task record, checks each claim against the live codebase, and applies corrections at review gates.

---

## Enforcement

These rules are enforced by:
1. **Flows** — Check rules at each phase gate
2. **Validators** — Run automated checks where possible
3. **Developer review** — Human verifies at [A]/[F]/[R] gates
4. **Knowledge base** — Violations recorded in lessons-learned

**Severity levels:**
- **error** — Must fix before proceeding (Rule 1, 2, 4 for system tasks, 6, 8, 9, 16)
- **warning** — Should fix, can proceed with justification (Rule 3, 4 for content tasks, 7, 10, 11, 12, 17, 18)
- **info** — Best practice, noted but not blocking (Rule 5, 13, 14, 15)
