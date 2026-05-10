# Flow: Orchestrate

> **Profile:** Web Frontend

## When to Use
After design-flow when the task has 3+ task flows with independent subsets and you want parallel implementation across them.

## Phases

This profile follows the canonical **4-phase, 2-gate** structure from `universal/workflow-structure.md` (and `profiles/generic/skeletons/orchestrate-flow.md`). Web-frontend notes below.

### Phase 1: PLAN *(STANDARD gate)*
- **1.1 LOAD / 1.2 GROUP / 1.3 SCHEDULE** — see canonical.
- **1.2 GROUP** — when validating "no two task flows in the same wave share files," watch for these web-frontend shared files:
  - `package.json` (dependency additions race)
  - `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock` (lockfile near-guaranteed conflict if any task flow adds/removes deps)
  - Route registration: `App.tsx`, `routes.ts`, `router.tsx`, `pages/_app.tsx`
  - Global state: store/context entry points (`store/index.ts`, `contexts/AppProvider.tsx`)
  - Style globals: `globals.css`, `tailwind.config.js`, theme files

→ **Gate 1: Execution plan review.**

### Phase 2: EXECUTE *(no gate)*
- **2.1 PREPARE WORKTREES** — `git worktree add .ai-workflow/worktrees/{task-name}/{task-flow}/ {current-branch}`. Each worktree needs its own `node_modules/` — see "node_modules strategy" below.
- **2.2 LAUNCH SUBAGENTS** — implement-flow in sub-agent mode (per Rule 16).
- **2.3 MONITOR / 2.4 WAIT FOR WAVE** — see canonical.

### Phase 3: MERGE *(CRITICAL gate)*
- **3.1 COLLECT / 3.2 CHERRY-PICK / 3.4 UPDATE RECORDS / 3.5 GENERATE REPORT** — see canonical.
- **3.3 VALIDATE MERGED** (web-frontend):
  ```bash
  npm test                # or: pnpm test, vitest run, jest
  npm run typecheck       # or: tsc --noEmit
  npm run lint            # or: eslint .
  ```
  After merge, run a fresh install (`npm ci`) to validate the merged lockfile resolves.

→ **Gate 2: Final implementation review.**

### Phase 4: COMMIT *(no gate)*
- See canonical. Cleanup loop:
  ```
  for wt in .ai-workflow/worktrees/{task-name}/*/; do
    git worktree remove "$wt"
  done
  ```

## node_modules strategy

`node_modules/` is gitignored and lives outside the worktree's tracked tree, so each worktree starts with no `node_modules/` after `git worktree add`. Three options for subagents:

1. **Fresh install per worktree** (safest, slowest). Each subagent runs `npm ci` first. Cost: N × install time + N × disk.
2. **Symlink shared `node_modules`** (fast, fragile). `ln -s ../../../node_modules .ai-workflow/worktrees/{task-name}/{task-flow}/node_modules` — works only if no subagent adds/upgrades a dependency. If any does, behavior diverges silently.
3. **pnpm with content-addressed store** (fast and safe). pnpm hard-links from a global store, so per-worktree installs are near-instant after the first.

Default to option 1 unless the project uses pnpm (option 3). Avoid option 2 unless the developer explicitly opts in.
