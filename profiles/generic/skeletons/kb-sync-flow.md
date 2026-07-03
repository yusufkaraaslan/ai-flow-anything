# Flow: KB Sync

> **Profile:** Generic  
> **Purpose:** Reconcile the knowledge base and task records with the project's current reality

---

## When to Use

- Any flow flagged a KB contradiction at a gate (Rule 18 load-time spot-check)
- After a stretch of work done outside the flows (hotfixes, manual edits, experiments)
- `--status` reported drifted task records or "never synced" KB files
- Before onboarding a new team member (they will trust the KB literally)
- On a regular cadence (e.g. end of each phase/milestone)

**Why this flow exists:** Rule 10 injects the KB as trusted context at the top of every flow run. A stale KB is worse than an empty one — wrong facts (an outdated test framework, a relaxed constraint still documented as hard, a phase label frozen at project start) poison every downstream decision. kb-sync is the active repair loop; Rule 18's load-time spot-check is the passive detector that tells you when to run it.

## Input

- Scope (optional): `project` | `team` | `tasks` | `all` (default `all`)
- Optionally a single task name to reconcile just that task's records

## Output

- Corrected KB files, updated **in place** (Rule 2 — no archives, no v2 copies)
- Reconciled task-record statuses and normalized status blocks
- `> **Last Synced:** {date}` stamps on verified project/team KB files
- A sync report (presented at the gate, summarized in the commit message)

## Prerequisites

- Knowledge base exists (`flow-storage/project/`, `flow-storage/team/`, `flow-storage/tasks/`)

---

## Phases

This flow has **2 review gates**: STANDARD after the sync diffs are applied, CRITICAL before commit. See `universal/workflow-structure.md` "KB Sync Flow Phases" for the canonical definition.

---

### Phase 1: SYNC

**Purpose:** Inventory every verifiable claim in scope, check each against the live project, reconcile task records, and apply corrections — one reviewable package.

**Sub-tasks (auto-proceed):**

**1.1 INVENTORY**
- Read every KB file in scope:
  - `flow-storage/project/*.md` (ARCHITECTURE, CONVENTIONS, PATTERNS, DECISIONS)
  - `flow-storage/team/*.md` (onboarding, workflows, glossary)
  - `.ai-workflow/rules.md` (project-specific rules)
  - `flow-storage/tasks/*/` task records — frontmatter + status blocks (scope `tasks` or `all`)
- Extract each **verifiable claim**: framework/library names, numeric constraints ("exactly N ..."), phase/status labels, documented commands, file paths, pattern references, directory structures.

**1.2 REALITY SCAN**
- Check each claim against the live project:
  - Referenced files/dirs still exist?
  - Named test framework matches what's actually installed (framework config, dev dependencies, test file imports)?
  - Documented commands still valid (binaries exist, paths resolve, script names current)?
  - Every pattern in PATTERNS.md still points at real code?
  - Numeric constraints match the current config/manifest?
  - Phase/status labels match the root README / CLAUDE.md / AGENTS.md?
- Profile overlays add tech-stack-specific reality signals — see the installed profile's `kb-sync-flow.md`.

**1.3 RECONCILE TASK RECORDS** *(scope `tasks` or `all`)*
- **Drifted statuses** — for each flow-plan file with `status: pending` or `in_progress`: do its declared Files to Create exist? Do its tests exist? Does `git log` reference it? If implemented, propose `status: complete` + `accepted-date` + a one-line note: "Reconciled by kb-sync-flow {date}: implementation verified in codebase."
- **Sign-off block integrity** — run the Rule 9 machine check on every `task-design.md`:
  ```
  grep -c '^> \*\*Status:\*\*' task-design.md         # must equal 1
  tail -n +10 task-design.md | grep -ci 'signed off'  # must equal 0
  ```
  Propose **formatting-only** normalization to the canonical block. If it's ambiguous whether a design was actually signed off, ask the developer at the gate — never guess the decision.
- **Misplaced artifacts** — files outside their spec'd subdirectory per `universal/knowledge-base-spec.md` (e.g. a technical design sitting in `implement/` instead of `design/`) → propose the move.
- **Superseded/duplicate tasks** — propose marking the stale one `Superseded by {task}` in its status block, cross-linked both ways. Never delete without an explicit developer instruction.

**1.4 PROPOSE & APPLY**
- Build the sync report: per file, **claim → observed reality → proposed edit** (as a diff)
- Apply the diffs in place (Rule 2)
- Stamp `> **Last Synced:** {date}` on each project/team KB file verified or corrected (add the line if missing)

**Hard constraints:**
- **Never edit the content of a signed-off task-design.md** (Rule 9). Formatting-only normalization of a malformed status block is the sole exception.
- Design deviations discovered during the scan go to `task-technical-design.md`, never `task-design.md`.
- **Never touch source code.** kb-sync fixes the *records* to match reality, not reality to match the records. If the scan reveals the *code* violates a real rule, report it at the gate as a finding for a separate free-flow/implement-flow run — do not fix it here.

**Artifacts (presented at gate):**
- Sync report (claim → reality → edit, per file)
- Applied diffs
- Ambiguous cases needing a human decision (unsure sign-offs, superseded tasks, code-vs-record conflicts)

**Gate:** Sync review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

---

### Phase 2: COMMIT

**Purpose:** Land the corrections. These become the trusted context every future flow run loads (Rule 10) — a wrong "correction" poisons future work just like the staleness it replaced.

**Sub-tasks (auto-proceed):**

**2.1 COMMIT** *(executes only after the gate is accepted; skipped with --no-commit)*
- Stage only KB and task-record files
- Compose commit message:
  ```
  docs(kb-sync): reconcile KB with codebase — {N} corrections

  Stale claims fixed: {list or count}
  Task records reconciled: {count}
  Last Synced stamped: {file count}
  ```
- Run `git commit`

**Gate:** Commit review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE C — CRITICAL, presented BEFORE git commit)*

**Failure modes to verify before [A]ccept:**
- Every change traceable to an observed reality (no speculative edits)?
- No source code staged (records only)?
- No signed-off task-design.md content changed (formatting-only normalization at most)?
- Ambiguous cases resolved by the developer, not guessed?
- `Last Synced` stamps present on touched project/team files?

---

## Sub-Agent Mode

When invoked by an orchestrator with `mode: sub-agent`, gates are suppressed (Rule 16). Return a structured report:

```markdown
## Sub-Agent Report: kb-sync ({scope})

**Status:** success | failure
**Claims checked:** {N}
**Stale claims found:** {N}
**Corrections applied:** {N}
**Task records reconciled:** {N}
**Ambiguous cases (need developer):** {list or "none"}
**Commit SHA:** {sha or "no commit"}
```

---

## Customization

Users can customize this flow by editing the generated copy in `.ai-workflow/flows/`:

- Add project-specific reality signals to check in 1.2 (e.g. "verify the services table against docker-compose.yml")
- Narrow the default scope (e.g. default to `project` if task records are managed elsewhere)
- Add a staleness budget (e.g. warn when `Last Synced` is older than 30 days)
