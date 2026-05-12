# Workflow Structure

> **Applies to:** ALL profiles  
> **Purpose:** Defines the flow architecture, phase patterns, and execution model

---

## Flow Types

Every profile provides one or more flows. The core set is:

| Flow | Purpose | Input | Output |
|--------|---------|-------|--------|
| **design-flow** | Design tasks with diagrams | Task idea | task-design.md, task-technical-design.md, task-edge-cases.md, diagrams, task flows |
| **implement-flow** | Implement task flows with validation | Task flow files | Source code, updated task flows (knowledge records) |
| **orchestrate-flow** | Parallel task flow orchestration | Task name (all task flows) | Merged implementation, consolidated report |
| **pr-flow** | PR lifecycle management | Completed task flows | Validation report, feedback task flows, lessons learned |
| **test-flow** | Test planning & execution | Task docs | Test files, coverage report |
| **deploy-flow** | Deployment & release | Build artifacts | Deployed release, verification report |
| **docs-flow** | Documentation maintenance | Existing docs | Updated docs, new patterns catalog |

Profiles may add custom flow types (e.g., `security-flow`, `perf-flow`).

---

## Phase Architecture

Every flow is composed of **phases**. Phases are sequential, each ending with a review gate (Rule 6).

**Phases vs sub-tasks.** Each phase typically contains multiple sub-tasks (numbered 1.1, 1.2, 1.3, ...). Sub-tasks **auto-proceed** as the agent works — no gate between them. Only the *parent phase* presents a gate when its sub-tasks are complete. This keeps Rule 6 satisfied (every phase ends with a gate) while limiting user-facing interruptions to the small number of meaningful checkpoints per flow.

The default cadence varies by flow type:

| Flow | Phase 1 gate | Phase 2 gate | Phase 3 gate | Phase 4 gate |
|------|-------------|-------------|-------------|-------------|
| design-flow | CRITICAL (only gate) | — (auto) | — | — |
| implement-flow | STANDARD | CRITICAL | — | — |
| orchestrate-flow | STANDARD (plan) | — (auto) | CRITICAL (merge) | — (auto) |
| pr-flow (all modes) | STANDARD | CRITICAL | — | — |
| test-flow | STANDARD | STANDARD | — | — |
| docs-flow | STANDARD | CRITICAL | — | — |
| deploy-flow | CRITICAL | CRITICAL | — | — |

Phases marked `— (auto)` execute automatically after the preceding gate is [A]ccepted — they apply decisions already reviewed and have no separate gate. This eliminates redundant "are you sure about that decision you just approved?" interruptions while keeping critical decision points where they matter.

### Design Flow Phases

The design flow uses **1 review gate** (CRITICAL) at the end of Phase 1. Phase 2 executes automatically after acceptance — its only purpose is to lock the design immutable and commit. A second gate there would re-review the already-approved content.

```
Phase 1: DESIGN
  Sub-tasks (auto-proceed):
    1.1 UNDERSTAND
        → Load project context (universal/, profiles/{detected}/,
          flow-storage/project/*)
        → Read existing docs and architecture
        → Explore codebase for related work
    1.2 DESIGN (Visual-First)
        → Generate class diagram, package diagram, sequence diagrams
          per universal/diagram-standards.md
        → Render every diagram source to .svg (or .png fallback) —
          both source and image are committed. An unrendered .puml
          is not a deliverable (Rule 4 — diagrams mandatory means
          *visible* diagrams).
        → If the local toolchain has no renderer (no `plantuml`,
          no Java+jar, no Docker), STOP. Tell the developer the
          install command for their OS (see diagram-standards.md
          "Local Rendering"). Resume after install + render succeeds.
          Do not present the gate with unrendered diagrams.
    1.3 SPECIFY
        → Write task-design.md, task-technical-design.md,
          task-edge-cases.md
        → Cross-reference diagrams from sub-task 1.2
    1.4 PLAN
        → Decompose task into implementation task flows
        → Each task flow gets frontmatter (task-flow, task, status,
          depends-on, accepted-date)
        → Each task flow declares Files to Create / Files to Modify
        → Build task flow dependency graph
        → Internal validation (surface failures at the Phase 1 gate
          so [F]eedback can revise):
            - Temporal contradiction: a task flow marked "Files to
              Modify" against a file no prior task flow has "Files
              to Create". Reorder or merge.
            - Type/symbol references: every type, enum, or class
              named in a Files list must be defined in this task's
              design or already in the codebase.
            - Coverage: scan task-design.md sections — is every
              public method, signal, and resource covered by exactly
              one task flow?
            - Pairwise file overlap: for every pair of task flows
              with NO depends-on edge between them, do their declared
              "Files to Create" / "Files to Modify" lists intersect?
              If yes, either add a depends-on edge (forcing them
              sequential) or merge them. This is verifiable at design
              time from the file lists alone — no `parallel-with`
              field is needed. The orchestrate-flow's GROUP sub-task
              will later enforce the same property at wave granularity,
              but catching it here means the developer fixes it once
              instead of bouncing through the wave plan.
            - Scope creep: any task flow larger than a one-day
              implementation? Split it.
        → Generate task flow dependency diagram
        → Stage flow-storage/tasks/{task-name}/ for preview
        → Compose proposed commit message

  [GATE TYPE C — CRITICAL]
  Design package review: developer sees the diagrams + spec docs +
  task flow files + staged diff + commit message together and forms
  one opinion. This is the ONE AND ONLY gate.
  
  ⚠️ Accepting locks task-design.md immutable per Rule 9 AND triggers
  Phase 2 (auto-executes FINALIZE + COMMIT). After this point, any
  change to task-design.md goes in task-technical-design.md instead.

  Failure modes to verify before [A]ccept:
    - PLAN integrity (any internal check failures from sub-task 1.4
      still unresolved?)
    - Anything you'd want to change in task-design.md? Now is the
      only time. Post-sign-off changes go in task-technical-design.md
      only, never task-design.md.
    - Diagrams cross-referenced correctly from task-design.md?
    - All diagrams rendered: every `.puml` (or `.d2` / `.mmd`) has a
      sibling `.svg` or `.png` in the same diagrams/ directory.
      Run: `for f in flow-storage/tasks/{task-name}/design/diagrams/*.puml; do
              base="${f%.puml}"; [ -f "${base}.svg" ] || [ -f "${base}.png" ]
              || echo "MISSING IMAGE: $f"
            done`
      Any "MISSING IMAGE" output means render before signing off.
    - All task flows from sub-task 1.4 actually written to disk?
    - Files staged: only flow-storage/tasks/{task-name}/ contents — no
      stray edits from other tasks or from .ai-workflow/. Run
      `git diff --cached --stat` and verify.
    - Commit message: task name correct? Task flow count matches
      what's in flow-plan/? Summary captures the *why* not the *what*?
    - Hooks won't fail (no linter complaint, no missing files).

  Opt-out: developer can pass --no-commit when invoking the flow
  to skip Phase 2's commit sub-task. The gate still presents, but
  FINALIZE still runs (design is locked even without commit).

Phase 2: LOCK & COMMIT
  (Auto-executes after Phase 1 [A]ccept. No separate review gate —
  the scope was already approved at Phase 1.)
  
  Sub-tasks:
    2.1 FINALIZE
        → Sign off task-design.md (status: v2.0 (SIGNED OFF),
          Immutable: Yes). This is a mechanical application of
          the decision already made at the Phase 1 gate.
    2.2 COMMIT
        → Run git commit with the message previewed at Phase 1.
          Skipped with --no-commit.
```

### Implement Flow Phases

```
Phase 1: BUILD
  Sub-tasks (auto-proceed):
    1.1 READ
        → Load architecture context (rules, profile, project docs)
        → Read task flow file (acceptance criteria, files to
          create/modify, deps)
        → Read task-design.md and task-technical-design.md sections
          relevant to this task flow
        → Find reference implementations in codebase
        → Hard refusal if no task-design.md or task-design.md not
          signed off — point developer at design-flow.
    1.2 PLAN
        → Analyze acceptance criteria against existing patterns
        → Structure the implementation (file order, function shape)
        → Surface ambiguities or design gaps
    1.3 IMPLEMENT
        → Write code incrementally per profile conventions
        → Apply universal rules and project rules
        → Cite reference implementations in comments only when
          non-obvious
    1.4 AUTO-VALIDATE
        → Run architecture/style checks specified in profile rules.md
        → Run the project's test suite (NEVER skip tests — user
          global rule)
        → Check integration points against task-design.md signal flow
        → If validation fails, do NOT auto-proceed. Surface failures
          and present them at the Phase 1 gate so developer can
          [F]eedback to fix or accept-with-known-failures.
    1.5 DOC-SYNC
        → Update task-technical-design.md with any deviations from
          task-design.md (Rule 9 — DESIGN stays immutable; deviations
          live in task-technical-design.md)
        → Update task-edge-cases.md with discoveries
        → Update flow-storage/project/PATTERNS.md with new reusable
          patterns

  [GATE TYPE B — STANDARD]
  Implementation review: developer sees files changed, validation
  results, doc updates together.

Phase 2: COMMIT
  Sub-tasks (auto-proceed):
    2.1 UPDATE TASK FLOW
        → Transform task flow file to knowledge record (status:
          complete, accepted-date set, implementation notes appended)
    2.2 COMMIT
        → Stage only the files this task flow produced
        → Compose commit message (conventional commits: feat/fix/docs/test)
        → Reference task flow name
        → Run git commit AFTER the gate is accepted

  [GATE TYPE C — CRITICAL]
  Commit review: presented BEFORE sub-task 2.2 executes git commit.
  ⚠️ Commit lands in history; amending later is friction.
  Failure modes to verify before [A]ccept:
    - Files staged: any unrelated stray edits sneaking in? Run
      `git diff --cached` mentally and verify each entry belongs.
    - Commit message: scope/type accurate? Description focuses on
      *why* not *what*?
    - Co-author trailer present if required by project policy?
    - Hooks won't fail (e.g. linter clean, no test failures)?
    - Pre-existing failing tests not silently bypassed?

  Opt-out: --no-commit skips sub-task 2.2 (gate still presents).
```

### Orchestrate Flow Phases

The orchestrate flow processes an entire task's task flows in parallel waves ordered by dependencies. Subagents implement individual task flows in isolated git worktrees; the orchestrator merges results and presents ONE gate at the end. Each subagent uses the implement-flow in **sub-agent mode** (no blocking gates, auto-proceed through all sub-tasks, return an implementation report).

```
Phase 1: PLAN
  Sub-tasks (auto-proceed):
    1.1 LOAD
        → Read all task flow files from
          flow-storage/tasks/{task-name}/implement/flow-plan/
        → Read task-design.md, task-technical-design.md,
          task-edge-cases.md
        → Read ARCHITECTURE.md, CONVENTIONS.md, PATTERNS.md
    1.2 GROUP
        → Build dependency graph from task flow frontmatter
          (depends-on field)
        → Topological sort → compute dependency waves
        → Group task flows by wave (all task flows in a wave
          are independent and can run in parallel)
    1.3 SCHEDULE
        → Generate execution plan:
          ```
          Wave 1 (parallel, 2 task flows):
            - user-session-store
            - auth-interfaces
          Wave 2 (parallel, 2 task flows):
            - auth-service          (blocked by: user-session-store)
            - login-form            (blocked by: auth-interfaces)
          Wave 3 (1 task flow):
            - api-integration       (blocked by: auth-service, login-form)
          Wave 4 (1 task flow):
            - tests                 (blocked by: all)
          
          Worktrees: .ai-workflow/worktrees/{task-name}/{task-flow}/
          ```
        → Validate wave plan: no two task flows in the same wave
          share files (from PLAN validation; if they do, mark one
          dependent on the other)

  [GATE TYPE B — STANDARD]
  Execution plan review: developer sees the grouped plan with
  dependency waves. Adjust grouping, reorder, or add/remove task
  flows via [F]eedback before execution begins.

Phase 2: EXECUTE
  Sub-tasks (auto-proceed):
    For each wave (waves run sequentially):
      2.1 PREPARE WORKTREES
          → For each task flow in this wave, create a git worktree:
            git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {base-branch}
          → Do NOT copy context files. The worktree is a clean
            checkout of {base-branch} containing every branch-tracked
            file already (design docs, KB files, .ai-workflow/).
            Only intervene if .ai-workflow/ is gitignored — see the
            profile's orchestrate-flow.md for the symlink fallback.
      2.2 LAUNCH SUBAGENTS (parallel)
          → For each task flow in this wave, launch one subagent
            running implement-flow in sub-agent mode.
          → Subagent receives:
            - Task flow file path + full content
            - Design docs relevant to this task flow
            - Mode: sub-agent (auto-proceed, no [A]/[F]/[R] gates)
            - Worktree path: .ai-workflow/worktrees/{task-name}/{task-flow}/
            - Return format: structured implementation report
          → Subagent follows implement-flow phases:
            READ → PLAN → IMPLEMENT → AUTO-VALIDATE → DOC-SYNC →
            UPDATE TASK FLOW → COMMIT (in worktree)
            All with auto-proceed — no user-facing gates.
          → Subagent returns: status, files created/modified,
            commit SHA, test results, deviations, issues.
      2.3 MONITOR
          → Track each subagent's progress
          → On failure: surface error to developer, decide
            retry / skip / abort (ask developer for non-trivial
            failures)
      2.4 WAIT FOR WAVE
          → All subagents in this wave must complete before
            the next wave starts.
          → Collect implementation reports + commit SHAs.

  [No gate between waves — waves auto-advance. The developer can
   interrupt at any time but no blocking gates in this phase.]

Phase 3: MERGE
  Sub-tasks (auto-proceed):
    3.1 COLLECT
        → Gather implementation reports from all subagents
        → Gather commit SHAs from all worktrees
        → Aggregate key findings: test results, deviations,
          issues encountered
    3.2 MERGE WORKTREES
        → For each worktree commit (in dependency order),
          cherry-pick into the main branch:
            git cherry-pick <sha> (from worktree)
        → Resolve merge conflicts:
          - Auto-resolve trivial conflicts where safe
          - Surface non-trivial conflicts for developer review
          - Document every conflict resolution
    3.3 VALIDATE MERGED
        → Run full test suite on the merged tree
        → Run integration checks across task flow boundaries
        → Verify no regressions against baseline
        → Check that all acceptance criteria are addressed
    3.4 UPDATE RECORDS
        → Transform all task flow files into knowledge records:
          status: complete, accepted-date set, implementation
          notes appended
        → Update task-technical-design.md with deviations
        → Update task-edge-cases.md with discoveries
        → Update PATTERNS.md with new patterns
    3.5 GENERATE REPORT
        → Consolidated implementation report:
          - Per-task-flow summary (status, files, commits, tests)
          - Merge conflicts resolved (with resolutions)
          - Validation results (pass/fail across all task flows)
          - Deviations from design
          - New patterns discovered
          - Issues requiring follow-up

  [GATE TYPE C — CRITICAL]
  Final implementation review: developer sees the merged code,
  the consolidated report, validation results, and all knowledge
  records. This is the ONE AND ONLY gate after execution.
  
  ⚠️ This is the single decision point for the entire feature
  implementation. Every task flow was implemented by a subagent
  in a worktree; their work has been cherry-picked into the
  main branch. Accepting finalizes the merge.

  Failure modes to verify before [A]ccept:
    - All task flows status: complete with accepted-date set?
    - Merge conflicts all resolved (no `<<<<<<<` left in tree)?
    - Full test suite passes (no regressions)?
    - All acceptance criteria from task-design.md addressed?
    - task-technical-design.md updated with deviations?
    - task-edge-cases.md updated with discoveries?
    - PATTERNS.md updated with new patterns?
    - Consolidated report captures all key findings?
    - No stray files from worktrees left in working tree?

Phase 4: COMMIT
  (Auto-executes after Phase 3 [A]ccept. No separate review gate —
  the work was already reviewed and accepted at Phase 3.)

  Sub-tasks:
    4.1 STAGE
        → Stage all implementation files + updated knowledge records
    4.2 COMMIT
        → Final consolidation commit:
            feat({task-name}): merged parallel implementation — {N} task flows

            Task flows implemented: {list}
            Merge conflicts resolved: {count, if any}
            Tests: {passing}/{total}
        → Run git commit
        → Clean up worktrees (one per subagent, removed individually
          since `git worktree remove` takes one path at a time):
            for wt in .ai-workflow/worktrees/{task-name}/*/; do
              git worktree remove "$wt"
            done
  
  Opt-out: --no-commit skips 4.2; staged files remain uncommitted.
```

### Docs Flow Phases

```
Phase 1: APPLY
  Sub-tasks (auto-proceed):
    1.1 SCAN
        → Identify which docs are stale or missing (compare last
          modified vs recent commits / merged PRs / new tasks)
        → Build a list of docs to update with rationale
    1.2 PROPOSE
        → For each doc to update, draft the change as a diff
        → Cross-check against living-document policy (Rule 2):
          update in place, no archives, no v2 copies
    1.3 APPLY
        → Apply each diff to its doc
        → Verify no broken cross-references introduced

  [GATE TYPE B — STANDARD]
  Doc updates review: developer sees the scan, proposed diffs,
  and applied changes together.

Phase 2: COMMIT
  Sub-tasks (auto-proceed):
    2.1 COMMIT
        → Stage updated docs/ files (and any AGENTS.md / CLAUDE.md
          edits if those were part of scope)
        → Compose commit message:
            docs: update {scope} — {one-line summary}

            Files updated: {list}
        → Run git commit AFTER the gate is accepted

  [GATE TYPE C — CRITICAL]
  Commit review: presented BEFORE git commit executes.
  Failure modes to verify before [A]ccept:
    - Files staged: only docs touched by this run — no code edits
      sneaking in (docs-flow never touches code).
    - Living-document compliance: did the diffs update files in
      place, or did they create *-v2.md / archive copies? The
      latter violates Rule 2.
    - task-design.md untouched (Rule 9 — immutable).

  Opt-out: --no-commit skips sub-task 2.1 (gate still presents).
```

### Test Flow Phases

```
Phase 1: WRITE
  Sub-tasks (auto-proceed):
    1.1 READ
        → Load task design + technical design + existing test patterns
        → Read profile rules for the project's test framework
    1.2 PLAN
        → Map test scenarios to acceptance criteria
        → Identify edge cases from task-edge-cases.md
        → Build a test outline (unit / integration / e2e split)
    1.3 GENERATE
        → Generate test stubs from the plan
    1.4 IMPLEMENT
        → Fill in test bodies, fixtures, and helpers
        → Apply project test conventions

  [GATE TYPE B — STANDARD]
  Test code review: developer sees outline + stubs + filled tests
  together. No changes land yet.

Phase 2: VERIFY
  Sub-tasks (auto-proceed):
    2.1 VALIDATE
        → Run the test suite (NEVER skip tests — user global rule)
        → Capture coverage delta vs baseline
        → If failures, do NOT auto-proceed. Surface at Phase 2 gate.
    2.2 DOCUMENT
        → Append a test summary to flow-storage/tasks/{task-name}/test/
        → Update task-edge-cases.md if new edges were uncovered

  [GATE TYPE B — STANDARD]
  Results review: developer sees pass/fail, coverage delta, and
  any new edge cases. test-flow does not git commit (tests get
  committed via implement-flow or pr-flow alongside the code
  they cover).
```

### Deploy Flow Phases

```
Note: deploy-flow keeps two CRITICAL gates. Each represents a
distinct hard-to-reverse decision (going live; declaring success).
Bundling them would lose meaningful safety.

Phase 1: PREPARE
  Sub-tasks (auto-proceed):
    1.1 CHECKLIST
        → Verify: tests pass, lint clean, version bumped, changelog
          updated, no debug code in build, secrets not in artifacts
    1.2 VERSION
        → Tag release (semver per project convention)
        → Generate release notes
    1.3 BUILD
        → Build artifacts (binary / image / bundle / etc.)
        → Verify artifact size + signatures

  [GATE TYPE C — CRITICAL]
  Ready to deploy? Presented BEFORE sub-task 2.1 (the actual
  deploy) executes. ⚠️ This is the last reversible point.
  Failure modes to verify before [A]ccept:
    - All checklist items pass?
    - Version number correct (no accidental bump direction)?
    - Build artifact sane (size / format / contents)?
    - Rollback plan documented?

Phase 2: SHIP
  Sub-tasks (auto-proceed):
    2.1 DEPLOY
        → Push artifact to target environment
        → Run smoke tests
    2.2 VERIFY
        → Health checks against live endpoints
        → Compare metrics against baseline (latency, error rate)
        → Hold for soak window if profile rules require one
    2.3 ANNOUNCE
        → Post release notes to configured channels
        → Update changelog in repo (commit separately)

  [GATE TYPE C — CRITICAL]
  All clear? Presented after VERIFY before ANNOUNCE goes out.
  ⚠️ Announcement is public. Roll back instead if metrics regressed.
  Failure modes to verify before [A]ccept:
    - Health checks all green for the soak window?
    - Error rate within tolerance vs baseline?
    - Anyone reporting issues out-of-band?
    - Release notes accurate and link to changelog?
```

### PR Flow Modes

PR flow is **modal** — the developer picks a mode and the flow runs the 2-phase pattern within that mode. Each mode has its own Phase 1 (work) + Phase 2 (commit / report) split.

```
Mode: pre-pr (default)
  Phase 1: VALIDATE
    Sub-tasks (auto-proceed):
      1.1 Task flow completeness check
      1.2 Documentation check
      1.3 Code quality validation
      1.4 Cross-task-flow integration check
      1.5 Test execution
      1.6 Registration/integration check
    [GATE TYPE B — STANDARD]
    Validation review: developer sees PASS/FAIL across all checks.

  Phase 2: REPORT
    Sub-tasks (auto-proceed):
      2.1 Generate the PR description from task flows + commits
      2.2 Open the PR (or print the description if --dry-run)
    [GATE TYPE C — CRITICAL]
    PR submission review: presented BEFORE the PR is opened.
    ⚠️ PR is publicly visible to reviewers once opened.
    Failure modes to verify before [A]ccept:
      - All Phase 1 validation checks PASS?
      - PR description captures the *why* and links task flows?
      - Branch is up to date with the target?
    Opt-out: --dry-run prints description without opening.

Mode: feedback
  Phase 1: PROCESS
    Sub-tasks (auto-proceed):
      1.1 Initialize session
      1.2 Per-comment loop
          → Developer pastes PR comment
          → AI analyzes (file, line, category, severity)
          → AI creates feedback task flow file
          → AI suggests fix
          → Developer: [Y] apply / [N] skip / [M] manual
          → Update task flow to knowledge record
    [GATE TYPE B — STANDARD]
    Feedback review: developer sees all processed comments + the
    fixes applied / skipped / queued.

  Phase 2: COMMIT
    Sub-tasks (auto-proceed):
      2.1 Stage feedback task flow files + any code fixes applied
      2.2 Compose commit message + run git commit AFTER the gate
    [GATE TYPE C — CRITICAL]
    Commit review: presented BEFORE git commit.
    Failure modes:
      - All feedback task flows have a clear fix / skip / manual
        decision recorded?
      - Code fixes correspond 1:1 to feedback task flows?
    Opt-out: --no-commit skips sub-task 2.2.

Mode: capture
  Phase 1: ANALYZE
    Sub-tasks (auto-proceed):
      1.1 Read all task flow files
      1.2 Generate lessons learned
      1.3 Propose knowledge base updates
      1.4 Update task catalog
      1.5 Update living documents (PATTERNS.md, DECISIONS.md,
          flow-storage/tasks/{task-name}/docs/lessons-learned.md)
      1.6 Capture metrics
    [GATE TYPE B — STANDARD]
    Capture review: developer sees the proposed lessons,
    PATTERNS/DECISIONS updates, and metrics together.

  Phase 2: COMMIT
    Sub-tasks (auto-proceed):
      2.1 Stage updated flow-storage/project/PATTERNS.md,
          flow-storage/project/DECISIONS.md,
          flow-storage/tasks/{task-name}/docs/lessons-learned.md,
          and any other living documents touched
      2.2 Compose commit message:
            docs(capture): {task-name} lessons learned and patterns

            Patterns added: {list}
            ADRs added: {list}
          Run git commit AFTER the gate is accepted
    [GATE TYPE C — CRITICAL]
    Commit review: presented BEFORE git commit.
    Failure modes to verify before [A]ccept:
      - Files staged: only the docs touched by capture — no stray
        edits to task code or task-design.md (which is immutable).
      - PATTERNS.md/DECISIONS.md edits use established sections
        (no random new sections that break the doc structure).
      - Commit message lists every pattern/ADR added.
    Opt-out: --no-commit skips sub-task 2.2; capture artifacts
    stay uncommitted.
```

---

## Phase Gate Protocol

Not every phase needs the same level of review. A "I just read the project files" phase doesn't deserve the same ceremony as "I'm about to lock task-design.md immutable forever." Three gate types exist; each flow phase declares which one it uses.

### Gate type A — LIGHT (auto-proceed with summary)

Use for **context-loading** phases that don't produce reviewable artifacts (Phase 1 UNDERSTAND, Phase 1 READ in implement-flow, etc.). The agent reports what it loaded and proceeds without asking. The developer can interrupt at any time, but no blocking gate. Saves a click per flow run for a phase where the answer is almost always "yes."

```
Phase {N}: {Name} complete.

Loaded:
  - {file/source 1}
  - {file/source 2}
  - {file/source 3}

Key facts found:
  - {fact 1}
  - {fact 2}

Proceeding to Phase {N+1}: {Next Name}.
(Interrupt now if anything above is wrong.)
```

### Gate type B — STANDARD ([A]/[F]/[R])

Use for phases that produce **reviewable artifacts** the developer can scan in 30 seconds and form an opinion on (design-flow Phase 1 SPECIFY package, implement-flow Phase 1 BUILD, test-flow Phase 1 WRITE, docs-flow Phase 1 APPLY). The default for any phase that doesn't lock something irreversible.

```
Phase {N}: {Name} complete.

Produced:
  - {artifact 1 with one-line summary}
  - {artifact 2 with one-line summary}

Review and choose:
  [A]ccept — Proceed to next phase
  [F]eedback — Provide feedback, I'll revise
  [R]eject — Start this phase over

Your choice:
```

### Gate type C — CRITICAL ([A]/[F]/[R] + review checklist)

Use for phases that produce **permanent or hard-to-fix artifacts**: design-flow Phase 2 COMMIT (locks task-design.md immutable AND lands the commit), implement-flow Phase 2 COMMIT (commit lands in history), deploy-flow Phase 1 PREPARE (last reversible point) AND Phase 2 SHIP (announcement is publicly irreversible), pr-flow Phase 2 (any mode — opens PR / lands commit). The standard `[A]/[F]/[R]` choice but the prompt explicitly names the failure modes the developer should look for. Prevents reflex-yes by giving the eye specific things to check.

```
Phase {N}: {Name} complete.

Produced:
  - {artifact 1}
  - {artifact 2}

⚠️  Before accepting, verify:
  - {failure mode 1 specific to this phase}
  - {failure mode 2}
  - {failure mode 3}

These are the things that bite later if you accept silently. Phase-specific
checklists are listed under each phase below.

Review and choose:
  [A]ccept — Proceed (this artifact may be hard to undo)
  [F]eedback — Provide feedback, I'll revise
  [R]eject — Start this phase over

Your choice:
```

### Handle Feedback / Reject (all gate types)

If **Feedback**:
1. Developer provides specific feedback
2. AI revises based on feedback
3. Re-present with changes highlighted
4. Loop until Accept or Reject

If **Reject**:
1. Ask developer: "Start over from beginning of phase, or specific sub-step?"
2. Restart from specified point
3. Previous work is not lost (saved in temp state)

If **Accept** (or auto-proceed for LIGHT gates):
1. Mark phase complete
2. Save artifacts
3. Proceed to next phase

---

## Resume Logic

Flows support resuming across sessions.

**Detection:**
- Check task flow frontmatter for `status`
- Read all task flow files in `flow-storage/tasks/{task-name}/implement/flow-plan/`
- Find first task flow with `status: pending` or `status: in_progress`

**Resume message:**
```
Implementation Status: {task-name}

  model.md       complete   (2026-04-15)
  controller.md  in_progress
  view.md        pending    (blocked by: controller)
  tests.md       pending    (blocked by: model, controller)
  integration.md pending    (blocked by: all above)

Progress: 1/5 complete

Resuming: controller.md
```

---

## Status Mode

Every flow supports `--status` flag:

```
/flow-name {task-name} --status
```

**Output:**
- Read all relevant files without modifying
- Show progress summary
- Identify blockers
- Suggest next action

**No implementation in status mode.** Read-only.

---

## Custom Flow Types

Profiles and users can define custom flow types.

**Requirements for a new flow:**
1. Markdown file in `.ai-workflow/flows/` or profile `skeletons/`
2. Clear purpose statement
3. Defined phases with gates
4. Specified input/output artifacts
5. Review protocol at each gate

**Example custom flow:**
```markdown
# Flow: Security Audit

## Purpose
Run security checks on new or modified tasks.

## When to Use
After implement-flow, before PR.

## Phases

(Gate cadence: see the gate table under "Phase Architecture" above.)

### Phase 1: AUDIT *(STANDARD gate)*
Sub-tasks (auto-proceed):
- 1.1 SCAN — search for SQL injection risks, exposed secrets, missing input validation, auth middleware gaps
- 1.2 ASSESS — categorize findings by severity, map to OWASP Top 10, check project security requirements
- 1.3 REPORT — generate security report, suggest fixes with code examples
[GATE 1: Findings + report review]

### Phase 2: COMMIT *(CRITICAL gate)*
Sub-tasks (auto-proceed):
- 2.1 Create security task flow files (if developer wants fixes addressed)
- 2.2 COMMIT — stage report + task flows; commit `security: audit + report`
[GATE 2: Commit review — presented BEFORE git commit]
```

---

## Artifact Flow

### Manual sequential path (implement-flow)

```
Developer idea
    ↓
Run design flow for {name}
    ↓
task-design.md + diagrams (class, package, sequence) + task flow files
    ↓
Run implement flow for {name}, task-flow=model
    ↓
Model implementation + tests → knowledge record
    ↓
Run implement flow for {name}, task-flow=controller
    ↓
Controller implementation + tests → knowledge record
    ↓
... (repeat for all task flows)
    ↓
Run PR flow for {name}, mode=pre-pr
    ↓
Validation report (PASS/FAIL)
    ↓
Developer creates PR
    ↓
Reviewers comment
    ↓
Run PR flow for {name}, mode=feedback
    ↓
Feedback task flows → fixes → knowledge records
    ↓
PR merged
    ↓
Run PR flow for {name}, mode=capture
    ↓
lessons-learned.md + PATTERNS.md updates
    ↓
Task complete
```

### Parallel orchestrated path (orchestrate-flow)

```
Developer idea
    ↓
Run design flow for {name}
    ↓
task-design.md + diagrams + task flow files (with dependency graph)
    ↓
Run orchestrate flow for {name}
    ↓
Phase 1: PLAN → execution plan gate
    ↓
Phase 2: EXECUTE
    ├─ Wave 1 (parallel): model + interfaces     [worktree A] [worktree B]
    ├─ Wave 2 (parallel): controller + component  [worktree C] [worktree D]
    └─ Wave 3: tests                               [worktree E]
    ↓
Phase 3: MERGE → cherry-pick all worktrees → validate → consolidated report gate
    ↓
Phase 4: COMMIT → single consolidation commit
    ↓
Run PR flow for {name}, mode=pre-pr
    ↓
Validation report → PR → review → feedback → merge
    ↓
Run PR flow for {name}, mode=capture
    ↓
lessons-learned.md + PATTERNS.md updates
    ↓
Task complete
```

---

## Extending the Workflow

### Add a phase to existing flow

Edit the flow markdown file, add a new phase section following the same pattern.

### Add a new flow type

Create new markdown file in `.ai-workflow/flows/` with phases, gates, and artifacts.

### Modify review options

Change the gate prompt text, but keep the [A]/[F]/[R] structure.

### Add artifacts

Specify new output files in phase descriptions. Update knowledge-base-spec.md if needed.
