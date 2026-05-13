---
name: ai-flow-anything
description: Universal workflow generator. Detects your project type, asks about your needs, and generates tailored AI-assisted development flows (design, implement, PR, test, deploy, docs) with living documentation and knowledge base support. Zero installation — everything is markdown that the AI reads and executes.
argument-hint: [command] [args]
---

# ai-flow-anything (Claude Code wrapper)

This is the Claude Code packaging of ai-flow-anything. The full platform-neutral workflow lives in `instructions.md` (one directory above this file once installed). **Read it before acting on any ai-flow-anything intent.**

---

## Command alias table

When the developer uses one of these slash commands, treat it as the corresponding ai-flow-anything intent and follow the workflow described in `instructions.md`.

| Slash command | Intent |
|---------------|--------|
| `/ai-flow-anything init` | Initialize ai-flow-anything for this project |
| `/ai-flow-anything flow <name>` | Run a specific flow (`<name>` is one of: design, implement, pr, test, deploy, docs) |
| `/ai-flow-anything status` | Show project workflow status |
| `/ai-flow-anything kb <query>` | Search the knowledge base |
| `/design-flow <task>` | Run the design flow against `<task>` |
| `/implement-flow <task>` | Run the implement flow against `<task>` (sequential, one task flow at a time) |
| `/orchestrate-flow <task>` | Run the orchestrate flow against `<task>` (parallel, all task flows via subagents) — handled by the dedicated `.claude/commands/orchestrate-flow.md` + `.claude/skills/orchestrate-flow/SKILL.md` |
| `/pr-flow <task>` | Run the PR flow against `<task>` |
| `/test-flow <task>` | Run the test flow against `<task>` |
| `/deploy-flow <task>` | Run the deploy flow against `<task>` |
| `/docs-flow <task>` | Run the docs flow against `<task>` |

If the developer expresses one of these intents in natural language ("design a task called auth", "implement the next task flow", "orchestrate auth") instead of a slash command, treat it the same way.

**Orchestrate-flow is the only flow that ships its own per-flow Claude skill, custom subagent (`orchestrate-implementer` with `isolation: worktree`), and slash command.** That triple is what enables parallel-wave execution natively on Claude Code. The other flows run via this master skill plus the rendered files in `.ai-workflow/flows/`.

---

## What this wrapper does

This file exists only to:

1. Provide the YAML frontmatter Claude Code uses to discover and trigger the skill.
2. Translate the slash-command syntax above into the platform-neutral intents that `instructions.md` describes.

All workflow logic — detection, discovery, profile loading, flow generation, knowledge-base structure, review gates — lives in `instructions.md`. Do not duplicate it here.

## Init must persist plumbing, not just generate output

When the developer runs `/ai-flow-anything init`, the agent must follow `instructions.md` Step 7 in full. A successful Claude Code init produces **all four** of the following, not just the last two:

1. **All four wrapper files installed:**
   - `.claude/skills/ai-flow-anything/SKILL.md` (symlink or copy of `platforms/claude/SKILL.md`)
   - `.claude/skills/orchestrate-flow/SKILL.md` (per-flow skill for orchestrate)
   - `.claude/agents/orchestrate-implementer.md` (custom subagent with `isolation: worktree`)
   - `.claude/commands/orchestrate-flow.md` (slash command)
2. **Universal entry point reachable:** `.ai-workflow/instructions.md`, `.ai-workflow/universal/`, and `.ai-workflow/profiles/{detected}/`.
3. **Rendered flows:** `.ai-workflow/flows/*.md` and (if needed) `.ai-workflow/rules.md`.
4. **Knowledge base scaffold:** `flow-storage/project/`, `flow-storage/team/`, `flow-storage/tasks/` per `universal/knowledge-base-spec.md`.

Run `instructions.md` Step 8 (Verify Install) before reporting success. Specifically: the Agent tool's `subagent_type` parameter must list `orchestrate-implementer` after install, or orchestrate-flow can't dispatch parallel subagents.
