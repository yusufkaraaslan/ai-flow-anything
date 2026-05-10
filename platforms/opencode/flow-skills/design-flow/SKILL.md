---
name: design-flow
description: MANDATORY before writing any task code. Use when the developer says "design a task", "design X", "let's design X", "plan task X", or "I want to design Y". Produces task-design.md, task-technical-design.md, task-edge-cases.md, diagrams, and implementation task flows for a task in a single review phase with 1 gate [A]/[F]/[R]. Refuses to skip the gate. The full flow is in .ai-workflow/flows/design-flow.md — load that file and execute it. Without a signed-off task-design.md from this flow, no task code may be written (project Rule 9).
license: MIT
metadata:
  audience: developers
  workflow: task-development
  phase: design
---

# design-flow (OpenCode wrapper)

This skill runs the **design phase** of ai-flow-anything for a task. The full flow lives in `.ai-workflow/flows/design-flow.md` in the project. **Read that file and execute it phase by phase.**

## When to invoke

Whenever the developer expresses any of:

- "Design a task called X"
- "Let's design X"
- "Plan the task X"
- "I want to start working on X" (with no prior task-design.md)
- An implementation request for a task that has no task-design.md yet — run design-flow first, then implement-flow

## What this flow does

**1 review gate (CRITICAL).** Phase 2 auto-executes after acceptance. See `.ai-workflow/universal/workflow-structure.md` "Phase Architecture" and "Phase Gate Protocol" for the full spec.

### Phase 1: DESIGN *(GATE TYPE C — CRITICAL)*

Sub-tasks (auto-proceed):
- **1.1 UNDERSTAND** — Load project context (universal/, profile/, flow-storage/project/).
- **1.2 DESIGN** — Produce diagrams (class, package, sequence at minimum) per `.ai-workflow/universal/diagram-standards.md`. **Render every diagram source to `.svg` (or `.png` fallback) and commit the image alongside the source.** A `.puml` without a matching image is not a deliverable. If no renderer is installed locally, stop and instruct the developer to install one before continuing.
- **1.3 SPECIFY** — Write `flow-storage/tasks/{task-name}/design/{task-design,task-technical-design,task-edge-cases}.md`.
- **1.4 PLAN** — Decompose into implementation task flows in `flow-storage/tasks/{task-name}/implement/flow-plan/`. Run internal validation (temporal contradictions in deps, undefined types, coverage gaps, parallel task flows sharing files, oversized task flows). Stage for preview + compose commit message.

→ **Gate: Design + plan review** — developer sees diagrams + spec docs + task flow files + staged diff + commit message together and forms one opinion. This is the ONE AND ONLY gate. Accepting locks task-design.md immutable per Rule 9 AND triggers Phase 2 (auto-executes FINALIZE + COMMIT).

### Phase 2: LOCK & COMMIT *(auto-executes after Phase 1 [A]ccept — no gate)*

Sub-tasks:
- **2.1 FINALIZE** — Sign off task-design.md (Rule 9: immutable after sign-off). Mechanical application of the decision already made at Phase 1.
- **2.2 COMMIT** — Stage `flow-storage/tasks/{task-name}/` and run `git commit` with message `design({task-name}): sign off task-design.md and {N} task flows`. Skipped with `--no-commit`.

## Hard rules

- Do not write any task code from this flow. Implementation is the implement-flow's job.
- Do not auto-accept the Phase 1 gate (Rule 6). Always present `[A]/[F]/[R]` to the developer.
- Sub-tasks inside Phase 1 auto-proceed; they are NOT user gates.
- Read `.ai-workflow/universal/rules.md` and `.ai-workflow/rules.md` before starting; they're non-negotiable.

For full instructions read `.ai-workflow/flows/design-flow.md`.
