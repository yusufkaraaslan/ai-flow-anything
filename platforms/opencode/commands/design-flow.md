---
description: Design a task — task-design.md, task-technical-design.md, diagrams, and implementation task flows in 1 review phase with a single CRITICAL gate. The COMMIT phase auto-executes after acceptance.
agent: build
---

Run the **design-flow** for: $ARGUMENTS

Load the `design-flow` skill (or read `.ai-workflow/flows/design-flow.md` directly if the skill is unavailable) and execute its phases in order.

### Phase 1: DESIGN *(CRITICAL gate)*
- **1.1 UNDERSTAND** — Load project context from `.ai-workflow/universal/`, `.ai-workflow/profiles/{detected}/`, `flow-storage/project/*`, and the project's design docs.
- **1.2 DESIGN** — Produce class, package, and sequence diagrams. Render every `.puml` to `.svg` (or `.png`); a source without an image is not a deliverable.
- **1.3 SPECIFY** — Write `flow-storage/tasks/{task-name}/design/{task-design,task-technical-design,task-edge-cases}.md`.
- **1.4 PLAN** — Decompose into implementation task flows in `implement/flow-plan/`. Run internal validation (temporal contradictions in deps, undefined types, coverage gaps, parallel task flows sharing files, oversized task flows). Stage for preview + compose commit message.
- → **Gate: Design + plan review** — diagrams + spec docs + task flow files + staged diff + commit message together. This is the ONE AND ONLY gate. `[A]ccept / [F]eedback / [R]eject`.

### Phase 2: LOCK & COMMIT *(auto-executes after Phase 1 [A]ccept — no gate)*
- **2.1 FINALIZE** — Sign off task-design.md (Rule 9: immutable after sign-off).
- **2.2 COMMIT** *(skipped if `--no-commit`)* — Stage `flow-storage/tasks/{task-name}/` and run `git commit` with message `design({task-name}): sign off task-design.md and {N} task flows`.

Hard rules:
- Do not auto-accept the Phase 1 gate (Rule 6). Always present `[A]/[F]/[R]` to the developer.
- Sub-tasks within Phase 1 auto-proceed; they are NOT user gates.
- Do not write any task code from this command — that's the implement-flow's job.
- Universal rules in `.ai-workflow/universal/rules.md` and project rules in `.ai-workflow/rules.md` are non-negotiable.

If $ARGUMENTS is empty, ask the developer for the task name (kebab-case) before proceeding.
