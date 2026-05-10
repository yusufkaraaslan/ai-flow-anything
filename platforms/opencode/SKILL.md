---
name: ai-flow-anything
description: Orchestrator skill for the ai-flow-anything task-development workflow. Use this skill ONLY for (a) initializing ai-flow-anything in a new project, (b) showing project workflow status across in-flight tasks, (c) searching the project knowledge base, or (d) when the developer's request is ambiguous about which phase flow applies. For specific phase work — design, implement, orchestrate, test, PR validation, deploy, docs — prefer the dedicated per-flow skills (design-flow, implement-flow, orchestrate-flow, pr-flow, test-flow, deploy-flow, docs-flow). ai-flow-anything enforces design-before-code: no task code without a signed-off task-design.md.
license: MIT
metadata:
  audience: developers
  workflow: task-development
  role: orchestrator
---

# ai-flow-anything (OpenCode wrapper)

This is the OpenCode packaging of ai-flow-anything. The full platform-neutral workflow lives in `instructions.md` at the ai-flow-anything installation root (typically `.ai-workflow/instructions.md` in the user's project). **Read it before acting on any ai-flow-anything intent.**

If `instructions.md` is not reachable in the user's project (no `.ai-workflow/instructions.md`, no symlink), the install is incomplete — the previous init only generated rendered output and skipped the install plumbing. In that case, before doing anything else, perform Step 7 of `instructions.md` against the ai-flow-anything source repo this wrapper is shipped from, then continue. Do not fall back to "improvise from the flow files alone" — the universal rules are not in those files.

---

## When to apply this skill

OpenCode loads this skill on demand based on the `description` above. Typical triggers:

- "Initialize ai-flow-anything for this project"
- "Design a task called <name>"
- "Implement the next task flow for <task>"
- "Orchestrate all task flows for <task>"
- "Validate <task> for PR"
- "Process PR feedback for <task>"
- "Capture lessons learned for <task>"
- "Search the knowledge base for <query>"

If the developer expresses any of these intents, translate it to the corresponding ai-flow-anything action and follow `instructions.md`.

---

## What this wrapper does

This file exists only to:

1. Provide the YAML frontmatter OpenCode uses to discover and load the skill on demand.
2. Point the agent at `instructions.md` for the full workflow logic.

All workflow logic — detection, discovery, profile loading, flow generation, knowledge-base structure, review gates — lives in `instructions.md`. Do not duplicate it here.

## Init must persist plumbing, not just generate output

When the developer says "initialize ai-flow-anything for this project", the agent must follow `instructions.md` Step 7 in full. Specifically: a successful OpenCode init produces **all four** of the following, not just the last two:

1. **This wrapper installed:** `.opencode/skills/ai-flow-anything/SKILL.md` (symlink or copy of `platforms/opencode/SKILL.md`).
2. **Universal entry point reachable:** `.ai-workflow/instructions.md`, `.ai-workflow/universal/`, and `.ai-workflow/profiles/{detected}/` — either as files (Layout A) or via a symlink at `.ai-workflow/` pointing at the ai-flow-anything clone (Layout B).
3. **Rendered flows:** `.ai-workflow/flows/*.md` and (if needed) `.ai-workflow/rules.md`.
4. **Knowledge base scaffold:** `flow-storage/project/`, `flow-storage/team/`, `flow-storage/tasks/` per `universal/knowledge-base-spec.md`.

If any of these is missing at the end of init, the install is broken and OpenCode will not be able to re-enter the workflow on the next session. Run `instructions.md` Step 8 (Verify Install) before reporting success.
