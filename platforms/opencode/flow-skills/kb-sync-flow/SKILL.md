---
name: kb-sync-flow
description: Use when the developer says "sync the KB", "sync the knowledge base", "the KB is stale", "reconcile task statuses", "KB says X but the project is Y", or when any flow flagged a KB contradiction (Rule 18). Audits every knowledge-base claim and task record against the live codebase in 2 phases with 2 review gates [A]/[F]/[R] (one after sync diffs are applied, one before commit) — corrects stale entries in place, reconciles drifted flow-plan statuses, normalizes malformed sign-off blocks, and stamps Last Synced dates. Never edits signed-off task-design.md content (Rule 9) and never touches source code. The full flow is in .ai-workflow/flows/kb-sync-flow.md.
license: MIT
metadata:
  audience: developers
  workflow: task-development
  phase: kb-sync
---

# kb-sync-flow (OpenCode wrapper)

This skill reconciles the knowledge base with the project's current reality. Full flow at `.ai-workflow/flows/kb-sync-flow.md`.

## When to invoke

- "Sync the KB" / "sync the knowledge base"
- "The KB is stale" / "KB says GUT but we use gdUnit4"
- "Reconcile task statuses" / "these flow-plans say pending but the feature shipped"
- Any flow's Rule 18 load-time spot-check flagged a contradiction
- `--status` reported drift or "never synced" KB files

## What this flow does

**2 phases, 2 gates.** Each phase contains sub-tasks that auto-proceed without intermediate gates. See `.ai-workflow/universal/workflow-structure.md` "KB Sync Flow Phases".

### Phase 1: SYNC *(GATE TYPE B — STANDARD)*

Sub-tasks (auto-proceed):
- **1.1 INVENTORY** — Read all KB files in scope (`project` | `team` | `tasks` | `all`); extract every verifiable claim.
- **1.2 REALITY SCAN** — Check each claim against the live codebase (test framework, numeric constraints, phase labels, commands, pattern references). Profile overlay lists tech-specific signals.
- **1.3 RECONCILE TASK RECORDS** — Drifted `status: pending` flow-plans vs actual code/tests/git; Rule 9 sign-off block machine check; misplaced artifacts; superseded duplicates.
- **1.4 PROPOSE & APPLY** — Sync report (claim → reality → edit); apply diffs in place; stamp `Last Synced`.

→ **Gate 1: Sync review** — report + diffs + ambiguous cases needing a human decision.

### Phase 2: COMMIT *(GATE TYPE C — CRITICAL)*

- **2.1 COMMIT** *(executes only after the gate is accepted)* — Stage records only; `docs(kb-sync): reconcile KB with codebase — {N} corrections`. Opt-out `--no-commit`.

→ **Gate 2: Commit review** — presented BEFORE git commit. These corrections become trusted context for every future flow run (Rule 10) — verify every change traces to observed reality.

## Hard rules

- **Signed-off task-design.md content is never edited** (Rule 9) — formatting-only normalization of a malformed status block is the sole exception.
- **Never touch source code** — kb-sync fixes the records to match reality. Code problems become findings for a separate flow.
- **Living documents** (Rule 2) — update in place, never archive.
- **Never guess a sign-off decision** — ambiguous cases go to the developer at the gate.

For full instructions read `.ai-workflow/flows/kb-sync-flow.md`.
