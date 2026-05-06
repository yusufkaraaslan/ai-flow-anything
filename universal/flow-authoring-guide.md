# Flow Authoring Guide

> **Applies to:** Profile authors, advanced users  
> **Purpose:** How to write flow skeletons that AI assistants can execute  
> **See also:** For creating a full profile (structure, discovery, rules, + skeletons), use [`docs/how-to-create-profile.md`](../docs/how-to-create-profile.md). This guide covers only flow skeleton authoring.

---

## What is a Flow?

A flow is a markdown file that an AI assistant reads and follows to complete a complex, multi-step task. Think of it as a recipe that the AI cooks from.

**Key characteristics:**
- Written in natural language (prose, not code)
- Divided into sequential phases
- Each phase ends with a review gate
- Specifies input, output, and artifacts
- Contains instructions the AI interprets and executes

---

## Flow Structure

Every flow file follows this structure:

```markdown
# Flow: {Name}

## Purpose
One-paragraph description of what this flow does.

## When to Use
Conditions under which this flow should be invoked.

## Input
What the flow needs to start (files, context, prior work).

## Output
What the flow produces (files, code, docs).

## Prerequisites
What must be true before this flow can run.

## Phases

### Phase 1: {Name}
**Purpose:** What this phase accomplishes.

**Steps:**
1. Step description (natural language instruction)
2. Step description
3. Step description

**Artifacts:**
- File path and description

**Gate:** [A]ccept / [F]eedback / [R]eject  *(or LIGHT auto-proceed, or CRITICAL with checklist — see "Choosing a gate type" below)*

### Phase 2: {Name}
...

## Customization
How users can customize this flow for their project.

## Example
Walkthrough of a typical execution.
```

---

## Writing Phases

Every flow uses the **2-phase, 2-gate** default pattern (see `universal/workflow-structure.md`). Each phase contains multiple sub-tasks (numbered 1.1, 1.2, ...) that auto-proceed without intermediate gates. Only the parent phase has a user-facing gate.

### Phase Purpose

Start each phase with a clear purpose statement:

```markdown
### Phase 1: BUILD *(STANDARD gate)*

**Purpose:** Write the source code, validate, and sync docs as one
reviewable package.

**Sub-tasks (auto-proceed):**

**1.1 READ** — Load context, task flow file, design docs.
**1.2 PLAN** — Map acceptance criteria to existing patterns.
**1.3 IMPLEMENT** — Write the source code per project conventions.
**1.4 AUTO-VALIDATE** — Run the project test suite.
**1.5 DOC-SYNC** — Update task-technical-design.md with deviations.

→ **Gate 1: Implementation review** — files changed + validation results + doc updates.
```

### Phase Steps

Write steps as natural language instructions. The AI interprets them.

**Good:**
```markdown
**Steps:**
1. Read the task flow file to extract acceptance criteria
2. Find 1-2 reference implementations of similar tasks
3. Read the reference implementations to understand patterns
4. Create the new files following the same patterns
5. Ensure all acceptance criteria are met
```

**Bad:**
```markdown
**Steps:**
1. grep "AC" task-flow.md
2. ls src/features/
3. cat reference1.{{ext}}
4. cp template.{{ext}} new.{{ext}}
```

The AI decides HOW to execute. You describe WHAT needs to happen.

### Phase Artifacts

List all files created or modified in this phase:

```markdown
**Artifacts:**
- `{{source-dir}}/features/{name}/Model/{Name}Model.{{ext}}` — Domain model
- `{{source-dir}}/features/{name}/Controller/{Name}Controller.{{ext}}` — Business logic
- `flow-storage/tasks/{task-name}/implement/flow-plan/task-flow-{task-flow-name}.md` — Updated knowledge record
```

### Phase Gate

Every phase ends with a gate, but **not every gate is the same**. Use the right type for each phase to avoid rubber-stamping fatigue.

```markdown
**Gate:** {LIGHT | STANDARD | CRITICAL}
```

The full protocol for each gate type lives in `universal/workflow-structure.md` under "Phase Gate Protocol." Summary:

- **LIGHT** — Agent reports loaded context and proceeds. No `[A]/[F]/[R]` choice. The developer can interrupt at any time but isn't asked.
- **STANDARD** — `[A]/[F]/[R]` with a summary of artifacts produced.
- **CRITICAL** — `[A]/[F]/[R]` plus a phase-specific *failure-mode checklist* the developer should mentally tick before accepting. Reduces reflex-yes when the artifact is permanent or hard to undo.

### Choosing a gate type

| Situation | Gate type | Why |
|---|---|---|
| Phase loads files / discovers context, produces no permanent artifact | LIGHT | Saves a click; nothing to review. |
| Phase produces a reviewable document or diagram the developer can scan in <30s | STANDARD | Brief artifact summary is enough. |
| Phase produces something **permanent** (commit, immutable doc, signed-off design) | CRITICAL | Force the eye to specific failure modes; prevents reflex-yes. |
| Phase decomposes work into many sub-pieces (e.g., task flow planning) | CRITICAL | Decomposition errors compound through implementation. |

**Heuristic:** if you'd be annoyed to discover a problem 30 minutes after accepting this phase, use CRITICAL and write the checklist. If the cost of "oops" is a one-line fix, STANDARD is fine. If there's nothing to review, LIGHT.

### Writing CRITICAL gate checklists

A good checklist names the specific failure modes that have actually bitten this phase before, in concrete terms. Bad: "verify quality." Good: "A task flow marked 'Files to Modify' against a file no prior task flow 'Files to Create' — temporal contradiction. Reorder or merge."

Three to five items max. Long checklists become rubber-stamps just like soft gates.

---

## Writing Instructions

### Be Specific About Intent

Tell the AI what you're trying to achieve, not how to do it.

**Good:**
> "Ensure the model follows the project's naming conventions and uses the base class discovered in the slot catalog."

**Bad:**
> "Name the class PascalCase and extend BaseModel."

### Reference Context

Always tell the AI to load context first:

```markdown
## First Step: Load Context

Before doing anything else, read:
1. `flow-storage/project/ARCHITECTURE.md`
2. `flow-storage/project/CONVENTIONS.md`
3. `flow-storage/project/PATTERNS.md`

This provides architecture rules, framework patterns, coding standards, 
and reference implementations.
```

### Cite Examples

When describing patterns, tell the AI to find examples:

```markdown
**Step 2: Find Reference Implementations**

Search the codebase for similar tasks. Look for:
- Tasks with similar complexity
- Tasks in the same domain
- Recently completed tasks (more likely to follow current conventions)

Read 1-2 reference implementations to understand:
- Class structure
- Lifecycle patterns
- Event handling
- Testing approach
```

### Handle Ambiguity

When something might vary, tell the AI to ask:

```markdown
**Step 3: Determine Async Pattern**

Check the codebase for async patterns:
- Search for `async/await` usage
- Search for Promise patterns
- Search for callback patterns

If multiple patterns are found, ask the developer:
"This project uses both async/await and Promises. Which pattern 
should I use for this task?"
```

---

## Slot Usage

Slots are placeholders for project-specific values discovered during ai-flow-anything initialization.

### Referencing Slots

In skeletons, use `{{slot-name}}` notation:

```markdown
## Step 1: Create Model

Create `{{source-dir}}/Model/{{Task}}Model.{{ext}}` extending 
`{{base-model-class}}`.

Follow the pattern from `{{reference-implementation}}`.
```

### Slot Resolution

During generation, the AI replaces `{{slot-name}}` with discovered values:
- `{{source-dir}}` → `src/` or `Assets/Scripts/` or `lib/`
- `{{base-model-class}}` → `BaseModel` or `AbstractEntity`
- `{{reference-implementation}}` → `UserModel.{{ext}}` (varies by language)

### Fallback

If a slot is empty, the AI asks the developer:

```markdown
**Fallback:** If `{{base-model-class}}` is not discovered, ask:
"What is the base class for models in this project?"
```

---

## Rule Integration

Flows must reference rules from two sources:

### Universal Rules

Always apply rules from `universal/rules.md`:

```markdown
**Apply Universal Rules:**
- Documentation before code (Rule 1)
- Unlimited iterations (Rule 3)
- No auto-approval (Rule 6)
- Transparent execution (Rule 11)
```

### Profile Rules

Apply profile-specific rules:

```markdown
**Apply Profile Rules:**
- No framework imports in domain layer (from `profiles/{type}/rules.md`)
- Use project's async pattern
- Follow naming conventions from `CONVENTIONS.md`
```

### Custom Rules

Check for project-specific overrides:

```markdown
**Check Custom Rules:**
Read `.ai-workflow/rules.md` if it exists. Apply any project-specific 
rules defined there.
```

---

## Common Patterns

### Resume Logic

Support resuming across sessions:

```markdown
## Resume Logic

Before starting, check for existing progress:
1. Read all task flow files in `flow-storage/tasks/{task-name}/implement/flow-plan/`
2. Find first task flow with `status: pending` or `status: in_progress`
3. If found, ask developer: "Resume from {task-flow-name}?"
4. If developer confirms, skip completed task flows

**Status display:**
```
Implementation Status: {task-name}

  model.md       complete   (2026-04-23)
  controller.md  in_progress
  view.md        pending    (blocked by: controller)

Progress: 1/3 complete
```
```

### Error Handling

Handle failures gracefully:

```markdown
## Error Handling

If a step fails:
1. Report the failure to developer with context
2. Suggest alternatives
3. Ask: "Continue with alternative, retry, or abort?"

**Never silently skip errors.**
```

### Progress Tracking

Show progress through phases:

```markdown
## Progress Display

After each phase, show:
```
Phase {N}: {Name} — COMPLETE ✓

Artifacts:
  - {file-path-1}
  - {file-path-2}

Next: Phase {N+1}: {Next Name}
```
```

---

## Testing Flows

Before publishing a profile, test the flow:

### Self-Review Checklist

- [ ] Each phase has a clear purpose
- [ ] Steps are natural language, not pseudo-code
- [ ] Each phase ends with a gate
- [ ] Artifacts are listed
- [ ] Resume logic is described
- [ ] Error handling is described
- [ ] Context loading is first step
- [ ] Rules are referenced
- [ ] Slots are used for project-specific values
- [ ] Example walkthrough is provided

### Test Execution

1. Create a test project
2. Initialize ai-flow-anything for it
3. Run each flow on a sample task
4. Verify output matches expectations
5. Check that gates work correctly
6. Verify knowledge base is populated

---

## Example: Minimal Flow

```markdown
# Flow: Code Review

## Purpose
Review code changes for quality, conventions, and potential issues.

## When to Use
Before committing code or creating a PR.

## Input
- Changed files (staged or unstaged)
- Project conventions from `flow-storage/project/CONVENTIONS.md`

## Output
- Review report with findings
- Suggested fixes

## Prerequisites
- Git repository
- Changes to review

## Phases

(Example custom flow following the canonical 2-phase, 2-gate pattern.)

### Phase 1: REVIEW *(STANDARD gate)*
**Purpose:** Read the diff, run all checks, and produce a single review report.

**Sub-tasks (auto-proceed):**

**1.1 READ** — Show git diff or changed files; read relevant conventions; identify files needing review.

**1.2 REVIEW** — Check naming conventions, code style, common issues, test coverage, documentation updates.

**Artifacts (presented at gate):**
- Review scope summary
- Review report with findings

**Gate:** Findings review — [A]ccept / [F]eedback / [R]eject

### Phase 2: FIX *(STANDARD gate)*
**Purpose:** Apply approved fixes and re-validate.

**Sub-tasks (auto-proceed):**

**2.1 PROPOSE** — Present fix suggestions with before/after; developer chooses which to apply.

**2.2 APPLY** — Apply approved fixes; re-run checks.

**Artifacts (presented at gate):**
- Fixed code
- Updated review report

**Gate:** Final approval — [A]ccept / [F]eedback / [R]eject

**Gate:** Confirm fixes — [A]ccept / [F]eedback / [R]eject
```

---

## Advanced: Conditional Logic

Flows can have conditional behavior based on project state:

```markdown
## Conditional: Task Type

If task involves data models:
  - Include ERD diagram
  - Check database migrations

If task involves UI:
  - Include accessibility checks
  - Check responsive design

If task involves API:
  - Include endpoint documentation
  - Check auth requirements
```

The AI evaluates conditions and adapts the workflow.

---

## Best Practices

1. **Start simple.** A flow with 3 clear phases is better than 8 confusing ones.
2. **Be explicit about gates.** Never let the AI proceed without developer input.
3. **Reference existing docs.** Don't duplicate information from CONVENTIONS.md or PATTERNS.md.
4. **Use examples.** Show the AI what good output looks like.
5. **Handle edge cases.** What if there's no reference implementation? What if the developer rejects?
6. **Keep it human-readable.** Other developers will read these files.
7. **Test incrementally.** Write one phase, test it, then add the next.
