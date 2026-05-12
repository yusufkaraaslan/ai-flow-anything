# Flow: Orchestrate

> **Profile:** Generic  
> **Purpose:** Orchestrate parallel implementation of all task flows for a task using isolated git worktrees and subagents.

---

## When to Use

After design-flow has completed and all task flows are planned with their dependency graph. Use orchestrate-flow to implement the entire task in parallel waves instead of running implement-flow once per task flow.

**Choose orchestrate-flow when:**
- The task has 3+ task flows with independent subsets
- You want to minimize wall-clock time by parallelizing independent work
- You want a single consolidated review gate for the entire feature

**Choose implement-flow (sequential) when:**
- The task has 1-2 task flows with simple dependencies
- You want granular per-task-flow review and commits
- The task flows are tightly coupled and benefits from sequential context

## Input

- Task name (e.g., `user-authentication`)
- All task flow files in `flow-storage/tasks/{task-name}/implement/flow-plan/`
- task-design.md, task-technical-design.md (must be signed off per Rule 9)

## Output

- Implemented source code (merged from all worktrees)
- All task flow files transformed to knowledge records
- Consolidated implementation report
- Single consolidation commit (or per-task-flow commits cherry-picked)

## Prerequisites

- Design flow completed and signed off (task-design.md immutable per Rule 9)
- Task flow files exist with dependency graph (depends-on frontmatter)
- Git worktree support available (`git worktree --help`)
- **Working directory clean.** Run `git status`. If anything is staged or unstaged, STOP and offer the developer:
  > Your working directory has uncommitted changes. orchestrate-flow's MERGE phase cherry-picks worktree commits into the main branch — that step is unsafe with a dirty WD (cherry-pick may refuse, may silently tangle your changes with the orchestrator's, or may overwrite work on `git checkout`).
  >
  > Options:
  >   [S]tash — `git stash push -m "pre-orchestrate {task-name}"`; resume after MERGE accept; you re-apply with `git stash pop`.
  >   [C]ommit — commit your in-flight work first, then run orchestrate-flow.
  >   [A]bort — exit; clean up manually before re-running.
  
  Do not auto-proceed. Wait for explicit choice.

---

## Phases

This flow has **2 review gates**: Phase 1 (STANDARD — plan review) and Phase 3 (CRITICAL — merged implementation review). Phases 2 and 4 auto-execute.

---

### Phase 1: PLAN

**Purpose:** Load all task flows, build the dependency graph, group into parallel waves, and present an execution plan for developer approval.

**Sub-tasks (auto-proceed):**

**1.1 LOAD**
- Read all task flow files from `flow-storage/tasks/{task-name}/implement/flow-plan/`
- Read `task-design.md`, `task-technical-design.md`, `task-edge-cases.md`
- Read `flow-storage/project/{ARCHITECTURE,CONVENTIONS,PATTERNS}.md`
- Verify: task-design.md is signed off (Rule 9). If not, STOP — run design-flow first.

**1.2 GROUP**
- Extract `depends-on` from each task flow frontmatter
- Build a directed acyclic graph (DAG)
- Topological sort → compute dependency waves:
  - Wave 0: task flows with no dependencies (can start immediately)
  - Wave 1: task flows whose deps are all in Wave 0
  - Wave N: task flows whose deps are all in Waves 0..N-1
- Validate wave plan:
  - No two task flows in the same wave share files (if they do, mark one dependent on the other and re-sort)
  - Every task flow appears in exactly one wave
  - No cycles in dependency graph

**1.3 SCHEDULE**
- Generate execution plan display:
  ```
  Execution Plan: {task-name}
  Task flows: {N} total

  Wave 1 (parallel, {M} task flows):       [no dependencies]
    - {task-flow-a}    {worktree-path-a}
    - {task-flow-b}    {worktree-path-b}

  Wave 2 (parallel, {K} task flows):       [blocked by: {wave-1-names}]
    - {task-flow-c}    {worktree-path-c}
    - {task-flow-d}    {worktree-path-d}

  Wave 3 (sequential, 1 task flow):        [blocked by: {wave-2-names}]
    - {task-flow-e}    {worktree-path-e}

  Worktree base: .ai-workflow/worktrees/{task-name}/*/
  Subagent mode: implement-flow auto-proceed (no internal gates)
  ```
- Display files each task flow will create/modify (from frontmatter)

**Artifacts (presented at gate):**
- Execution plan with wave breakdown
- Per-task-flow file lists
- Worktree paths

**Gate:** Execution plan review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE B — STANDARD)*

Review the wave grouping. Adjust via [F]eedback: reorder waves, split a task flow, merge two small ones, or change which task flows run in parallel. [R]eject to start plan over.

---

### Phase 2: EXECUTE

**Purpose:** Create git worktrees and launch subagents for each wave, running parallel task flows within each wave. No user-facing gates — the plan was already approved.

**Sub-tasks (auto-proceed):**

For each wave (sequential — waves run in order):

**2.1 PREPARE WORKTREES**
- For each task flow in the current wave, create an isolated git worktree:
  ```
  git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {current-branch}
  ```
- `git worktree add` produces a clean checkout of `{current-branch}` containing every branch-tracked file. **Do not copy context files into the worktree** — they are already there. This includes `flow-storage/tasks/{task-name}/design/*`, `flow-storage/project/*`, and `.ai-workflow/` itself (which the project tracks per Layout A in `instructions.md` Step 7.2).
- **Exception — gitignored `.ai-workflow/`**: if the project gitignores `.ai-workflow/` (uncommon, against Layout A), the worktree won't have it. In that case STOP and ask the developer to either commit `.ai-workflow/` first or accept that subagents will need a symlink:
  ```
  ln -s ../../../../../.ai-workflow .ai-workflow/worktrees/{task-name}/{task-flow}/.ai-workflow
  ```
  (count the `../` carefully — the symlink target is relative from the worktree root)
- Verify each worktree is clean and on the correct branch (`git -C <worktree-path> status` should report "nothing to commit, working tree clean").

**2.2 LAUNCH SUBAGENTS (parallel)**
- For each task flow in the wave, launch one subagent.
- Subagent invocation (platform-specific; the canonical prompt is below):

  ```
  You are a sub-agent of the implement-orchestrator for task "{task-name}".
  Your job: implement ONE task flow using implement-flow in sub-agent mode.

  Task flow file:
    flow-storage/tasks/{task-name}/implement/flow-plan/{task-flow-file}.md

  Mode: sub-agent
    - Follow the implement-flow phases (READ → PLAN → IMPLEMENT →
      AUTO-VALIDATE → DOC-SYNC → UPDATE TASK FLOW → COMMIT) but
      auto-proceed through ALL sub-tasks.
    - Do NOT present [A]/[F]/[R] gates. The orchestrator handles
      the only gate (Phase 3 MERGE).
    - Work in your git worktree at:
      .ai-workflow/worktrees/{task-name}/{task-flow}/
    - Commit in your worktree when done.
    - Return a structured implementation report.

  Implementation report format:
    ## Sub-Agent Report: {task-flow} ({task-name})
    **Status:** success | failure
    **Commit SHA:** <sha>
    **Files Created:** [...]
    **Files Modified:** [...]
    **Test Results:** {passing}/{total}, coverage delta: +X%
    **Deviations from Design:** [...]  (none if none)
    **Issues Encountered:** [...]     (none if none)
  ```

**2.3 MONITOR**
- Track each subagent's progress (display per-wave status)
- On subagent failure:
  - Surface the error + the subagent's report
  - Ask developer: retry / skip / abort entire orchestration?
  - Do not proceed to next wave until resolved

**2.4 WAIT FOR WAVE**
- All subagents in this wave must complete before the next wave starts
- Collect implementation reports + commit SHAs from all subagents in this wave
- Display wave completion summary before starting next wave

**Wave completion display:**
```
Wave 1 complete (2/2):
  ✓ user-session-store   commit a1b2c3d   tests: 8/8
  ✓ auth-interfaces      commit e4f5g6h   tests: 3/3

Starting Wave 2 (2 task flows)...
```

[No gate between waves. The developer can interrupt at any time but no blocking gates in this phase.]

---

### Phase 3: MERGE

**Purpose:** Collect all implementation results, cherry-pick worktree commits into the main branch, resolve conflicts, validate the merged tree, and generate a consolidated report for developer review.

**Sub-tasks (auto-proceed):**

**3.1 COLLECT**
- Gather all implementation reports from subagents
- Gather all commit SHAs from worktrees
- Aggregate key findings:
  - Total task flows completed / total
  - Total tests passing / total
  - Coverage delta
  - Deviations from design (across all task flows)
  - Issues encountered

**3.2 CHERRY-PICK WORKTREES**
- For each worktree commit, in dependency order (wave by wave):
  ```
  git cherry-pick <sha>
  ```
- If cherry-pick conflicts:
  - Auto-resolve trivial conflicts (identical additions, non-overlapping changes)
  - Document every automatic resolution
  - For non-trivial conflicts, STOP and surface to developer:
    ```
    Merge conflict in {file}:
      {task-flow-a} added {code-block} on lines {range}
      {task-flow-b} also modified {file} on lines {range}
    
    Options:
      [R]esolve manually — I'll show the conflict; you guide resolution.
      [S]kip {task-flow-b} — revert its cherry-pick and mark it for sequential re-implementation.
      [A]bort — stop merge, all worktrees preserved.
    ```

**3.3 VALIDATE MERGED**
- Run full test suite on the merged working tree
- Run integration checks across task flow boundaries
- Verify no regressions against baseline
- Check that all acceptance criteria from task-design.md are addressed
- Run architecture/style checks per profile rules

**3.4 UPDATE RECORDS**
- Transform all task flow files into knowledge records:
  ```yaml
  status: complete
  accepted-date: {ISO 8601 date}
  ```
- Append implementation notes to each task flow file:
  - What was implemented
  - Patterns used
  - Deviations from design
  - Commit SHA
  - Test results
- Update `task-technical-design.md` with all deviations (Rule 9)
- Update `task-edge-cases.md` with all discoveries
- Update `flow-storage/project/PATTERNS.md` with new patterns

**3.5 GENERATE REPORT**
- Consolidated implementation report:
  ```markdown
  # Orchestration Report: {task-name}
  
  ## Summary
  - Task flows implemented: {N}/{N}
  - Waves executed: {W}
  - Total commits: {C}
  - Tests: {passing}/{total} (coverage: +X%)
  - Merge conflicts resolved: {count}
  
  ## Per-Task-Flow Results
  
  ### {task-flow-a} (Wave 1)
  - Status: success
  - Commit: abc1234
  - Files created: [...]
  - Files modified: [...]
  - Tests: 8/8 passing
  - Deviations: none
  
  ### {task-flow-b} (Wave 1)
  - Status: success
  - Commit: def5678
  - Files created: [...]
  - Tests: 3/3 passing
  - Deviations: [1] Used factory pattern instead of dependency injection (noted in TDD)
  
  ... (all task flows)
  
  ## Merge Conflicts
  - {count} resolved (all trivial auto-resolutions)
  
  ## Patterns Added
  - {pattern-1}: {description}
  
  ## Deviations from Design
  - {deviation-1}: {resolution}
  
  ## Issues for Follow-Up
  - {issue-1}
  ```

**Artifacts (presented at gate):**
- Consolidated implementation report (above)
- All task flow files updated to knowledge records
- Updated task-technical-design.md, task-edge-cases.md, PATTERNS.md
- Staged file list (`git diff --cached --stat`)

**Gate:** Final implementation review — [A]ccept / [F]eedback / [R]eject  
*(GATE TYPE C — CRITICAL)*

⚠️ **This is the ONE AND ONLY implementation gate.** Every task flow was implemented by a subagent. Their work has been cherry-picked into the main branch. Accepting finalizes the entire feature implementation.

**Failure modes to verify before [A]ccept:**
- All task flows status: complete with accepted-date set?
- Merge conflicts all resolved (no `<<<<<<<` left in tree)?
- Full test suite passes (no regressions)?
- All acceptance criteria from task-design.md addressed?
- task-technical-design.md updated with all deviations?
- task-edge-cases.md updated with all discoveries?
- PATTERNS.md updated with new patterns?
- Consolidated report captures all key findings?
- No stray files from worktrees left in working tree?

---

### Phase 4: COMMIT

**Purpose:** Stage and commit all implementation files + updated knowledge records. No separate gate — the work was already reviewed and accepted at Phase 3.

**Sub-tasks (auto-execute after Phase 3 [A]ccept):**

**4.1 STAGE**
- Stage all implementation files + updated knowledge records
- Verify: only files from this task are staged (`git diff --cached --stat`)

**4.2 COMMIT**
- Run `git commit`:
  ```
  feat({task-name}): merged parallel implementation — {N} task flows in {W} waves

  Task flows implemented: {list}
  Waves: {W}
  Merge conflicts resolved: {count, if any}
  Tests: {passing}/{total}
  ```
- Clean up worktrees (one per subagent — `git worktree remove`
  takes one path at a time, so loop through them):
  ```
  for wt in .ai-workflow/worktrees/{task-name}/*/; do
    git worktree remove "$wt"
  done
  ```

**Opt-out:** `--no-commit` skips 4.2; staged files remain uncommitted.

---

## Platform Requirements

orchestrate-flow needs the host AI tool to support **two** things:

1. **Parallel subagent invocation** — the orchestrator must be able to spawn N subagents that run concurrently and return their reports independently.
2. **Per-subagent working directory** — each subagent must operate inside its own worktree path, isolated from the others. Otherwise all parallel subagents step on each other in the main project root and the worktree isolation is defeated.

| Tool | Parallel subagents | Per-subagent CWD | Status |
|---|---|---|---|
| Claude Code | Yes (Task tool, native parallel) | Yes (subagent operates in main CWD; pass worktree path as parameter) | Supported |
| OpenCode | Documented: parallel via "Task tool" + General subagent ([opencode.ai/docs/agents](https://opencode.ai/docs/agents/)) | **NOT documented.** Prototype before relying on it. | Experimental |
| Cursor / GitHub Copilot / Kimi-Code CLI | No primary subagent primitive | n/a | Not supported — use sequential implement-flow instead |

**Workaround when per-subagent CWD is not supported (or unverified):** instruct each subagent to prefix every git command with `git -C .ai-workflow/worktrees/{task-name}/{task-flow}/ ...` instead of relying on CWD. File reads/writes use the absolute worktree path. This is fragile (any tool call that uses `cwd`-relative paths breaks isolation) but works as a fallback.

**For OpenCode users:** prototype with two parallel subagents on a 2-task-flow task before running orchestrate-flow on real work. If the subagents step on each other's files, fall back to sequential implement-flow until OpenCode documents per-subagent CWD support.

---

## Resume Logic

If orchestrate-flow was interrupted (subagent crash, developer abort, session timeout):

1. Read all task flow files in `flow-storage/tasks/{task-name}/implement/flow-plan/`
2. Check for existing worktrees: `git worktree list | grep worktrees/{task-name}/`
3. Determine last completed wave by task flow status:

```
Orchestration Status: {task-name}

Wave 1 (2/2 complete):
  ✓ user-session-store    complete   commit a1b2c3d
  ✓ auth-interfaces       complete   commit e4f5g6h
Wave 2 (1/2 complete):
  ✓ auth-service          complete   commit i7j8k9l
  ✗ login-form             pending   (worktree intact)
Wave 3 (pending):          (blocked by: Wave 2)
  - api-integration        pending
Wave 4 (pending):          (blocked by: Wave 3)
  - tests                  pending

Progress: 3/5 complete

Resuming: login-form (in Wave 2, worktree intact)
Or: re-run orchestrate-flow from scratch (--restart)
```

**Resume strategy:**
- If all task flows are `status: complete`, skip to Phase 3 MERGE
- If a wave is partially complete, resume from the pending task flows in that wave
- If a worktree exists for a pending task flow, reuse it (don't recreate)
- If the main working tree has uncommitted cherry-picks from a prior partial merge, `git cherry-pick --abort` before resuming
- If `--restart`, remove all worktrees (`git worktree remove <path>` for each) and start from Phase 1

---

## Customization

Users can customize:
- Worktree base path (default: `.ai-workflow/worktrees/`)
- Max parallel subagents per wave
- Conflict resolution strategy (auto-resolve thresholds)
- Commit message format
- Report format