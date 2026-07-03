---
description: Reconcile the knowledge base and task records with the project's current reality
agent: build
---

Run the **kb-sync-flow** for: $ARGUMENTS

The argument is one of:
- `project` | `team` | `tasks` — limit the sync scope
- `<task-name>` — reconcile just that task's records
- (empty) — full sync (`all`): project KB + team KB + `.ai-workflow/rules.md` + all task records

Load the `kb-sync-flow` skill (or read `.ai-workflow/flows/kb-sync-flow.md` directly) and execute its **2 phases**. Each phase contains sub-tasks that auto-proceed without intermediate gates:

### Phase 1: SYNC *(STANDARD gate)*
- **1.1 INVENTORY** — Read every KB file in scope; extract each verifiable claim (frameworks, numeric constraints, phase labels, commands, pattern references).
- **1.2 REALITY SCAN** — Check each claim against the live codebase. Use the profile's reality-signal table.
- **1.3 RECONCILE TASK RECORDS** — Flag `status: pending` flow-plans whose implementation exists (propose `complete` + reconciliation note); run the Rule 9 sign-off machine check; find misplaced artifacts and superseded duplicates.
- **1.4 PROPOSE & APPLY** — Build the sync report (claim → observed reality → edit), apply diffs in place, stamp `> **Last Synced:** {date}`.
- → **Gate 1: Sync review** — report + diffs + ambiguous cases.

### Phase 2: COMMIT *(CRITICAL gate)*
- **2.1 COMMIT** *(executes only after the gate is accepted)* — Stage KB/task-record files only. Commit message: `docs(kb-sync): reconcile KB with codebase — {N} corrections`. Skip with `--no-commit`.
- → **Gate 2: Commit review** — presented BEFORE git commit.

Hard rules:
- **Never edit signed-off task-design.md content** (Rule 9) — formatting-only status-block normalization at most.
- **Never touch source code** — records match reality, not the other way around. Code problems become findings for a separate flow.
- **Never guess sign-off decisions** — ambiguous cases go to the developer at the gate.
