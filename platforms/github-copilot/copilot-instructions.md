# ai-flow-anything is installed in this repository

This project uses **ai-flow-anything**, a markdown-only AI workflow generator. The full workflow logic lives in `instructions.md` at the repo root (or under `.ai-workflow/` if installed there).

## When the developer asks about ai-flow-anything

If the developer's request matches one of these intents, read `instructions.md` and follow the workflow it describes. Do not improvise — ai-flow-anything has strict review gates and a defined phase architecture.

- Initialize ai-flow-anything for this project
- Design a task
- Implement the next task flow for a task
- Run a flow (design, implement, free, parallel-implement, pr, test, deploy, docs, kb-sync) against a task
- Validate a task for PR
- Process PR feedback
- Capture lessons learned for a task
- Search the project knowledge base

For all other requests, ignore this section.

## Init must persist plumbing, not just generate output

When the developer says "initialize ai-flow-anything for this project", the agent must follow `instructions.md` Step 7 in full. A successful Copilot init produces all four of: this file at `.github/copilot-instructions.md` (appended, not overwriting any existing content); universal entry point reachable at `.ai-workflow/{instructions.md, universal/, profiles/{detected}/}`; rendered flows at `.ai-workflow/flows/*.md`; and knowledge base scaffold at `flow-storage/{project,team,tasks}/`. Run `instructions.md` Step 8 (Verify Install) before reporting success.

## Why this is brief

GitHub Copilot loads `copilot-instructions.md` into every completion's context. Keeping this file minimal avoids bloating the context for everyday code completion. The full ai-flow-anything workflow lives in `instructions.md`, which is loaded only when relevant.
