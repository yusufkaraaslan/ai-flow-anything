# ai-flow-anything

> **You are the engine.** Read these markdown files, interpret them, and act according to their instructions. No external tools, no CLI, no DSL. Just markdown.

This file is the platform-neutral entry point for ai-flow-anything. Platform wrappers (in `platforms/{tool}/`) point at this file and translate platform-specific triggers (Claude Code skill descriptions, Cursor `.mdc` rules, GitHub Copilot instructions, OpenCode skills, Kimi Code `AGENTS.md`, etc.) into the intents listed below.

---

## Supported Actions

ai-flow-anything responds to the following developer intents. Each platform wrapper documents how to invoke them on its tool — but the underlying behavior is the same everywhere.

| Intent | What the AI does |
|--------|------------------|
| **Initialize ai-flow-anything for this project** | Detect project type, discover the codebase, ask the developer for missing context, and generate tailored flows into `.ai-workflow/flows/`. |
| **Run a specific flow** (design, implement, orchestrate, pr, test, deploy, docs) | Load the corresponding flow file from `.ai-workflow/flows/` and execute its phases against the named task. |
| **Show project workflow status** | Read the knowledge base and report progress across in-flight tasks and their task flows. |
| **Search the knowledge base** | Query `flow-storage/project/`, `flow-storage/team/`, and `flow-storage/tasks/` for relevant prior work. |

When the developer expresses one of these intents — whether through a slash command on a platform that supports them, a natural-language request, or any other channel — the AI follows the workflow described below.

---

## How It Works

### Step 0: Identify the Host AI Tool

Before anything else, identify **which AI CLI / IDE the developer is using to invoke ai-flow-anything**. This determines which platform wrapper from `platforms/{tool}/` must be installed in Step 7.

**Do not infer the tool from the model name.** "Kimi" and "Claude" are LLM providers; "OpenCode", "Claude Code", "Cursor", "Kimi-Code CLI", and "GitHub Copilot" are the *tools* — they each have their own discovery convention and install path. A single LLM (e.g. Kimi) can be used inside multiple tools, and each tool needs its own wrapper.

**Detection signals (check in this order):**

| Signal in project | Likely tool | Wrapper to install |
|---|---|---|
| `.opencode/` exists, or developer says "OpenCode" | OpenCode CLI | `platforms/opencode/SKILL.md` → `.opencode/skills/ai-flow-anything/SKILL.md` |
| `.claude/` exists, or developer says "Claude Code" | Claude Code | `platforms/claude/SKILL.md` → `.claude/skills/ai-flow-anything/SKILL.md` |
| `.cursor/` exists, or developer says "Cursor" | Cursor | `platforms/cursor/ai-flow-anything.mdc` → `.cursor/rules/ai-flow-anything.mdc` |
| `.github/copilot-instructions.md` exists, or developer says "Copilot" | GitHub Copilot | `platforms/github-copilot/copilot-instructions.md` → `.github/copilot-instructions.md` |
| Developer explicitly says "Kimi-Code CLI" (the tool, not the model) | Kimi-Code CLI | `platforms/kimi-code/AGENTS.md` → `AGENTS.md` |

**If none of these exist or are mentioned, ask the developer explicitly:**

> "Which AI tool will be invoking ai-flow-anything in this project?
> 1. Claude Code (CLI/IDE)
> 2. OpenCode CLI
> 3. Cursor IDE
> 4. GitHub Copilot
> 5. Kimi-Code CLI
> 6. Other / multiple — install more than one wrapper"

**If multiple tools are in use, install multiple wrappers** — they coexist without conflict, and the universal `instructions.md` + `universal/` + `profiles/` files are shared by all of them.

Record the chosen tool(s) in the slot catalog as `host-tool` (single) or `host-tools` (list).

### Step 1: Detect Project Type

Read all `profiles/*/README.md` files. Each profile contains detection hints. Score the current project against each profile.

**Detection strategy:**
1. **Auto-detect** — Scan project files, match against profile detection hints
2. **Ask developer** — If multiple profiles match with similar confidence, present options
3. **Create custom** — If no profile matches, ask: "Would you like to create a custom profile from scratch?"

**Example detection logic:**
- Unity: `Assets/` + `*.cs` + `ProjectSettings/` = high confidence
- Web: `package.json` + `src/` = high confidence  
- Godot: `project.godot` + `*.gd` = high confidence
- Generic: Any project (fallback, always matches with low confidence)

### Step 2: Load Universal Rules

Always read `universal/rules.md` first. These are non-negotiable guardrails that apply to ALL projects regardless of type.

### Step 3: Load Profile

Read the detected profile:
- `profiles/{type}/README.md` — Profile overview and capabilities
- `profiles/{type}/discovery.md` — Instructions for exploring this project type
- `profiles/{type}/rules.md` — Profile-specific conventions and guardrails
- `profiles/{type}/skeletons/*.md` — Flow templates to fill

### Step 4: Execute Discovery

Follow `profiles/{type}/discovery.md` instructions. The AI explores the codebase by:
- Reading key files (README, package manifests, config files)
- Searching for patterns (architecture, frameworks, conventions)
- Asking the developer about undiscoverable decisions

Record all findings in a slot catalog format.

### Step 5: Generate Flows

For each skeleton in `profiles/{type}/skeletons/`:
1. Load the skeleton markdown
2. Fill project-specific slots with discovered values
3. Apply universal rules + profile rules
4. Write to `.ai-workflow/flows/{flow-name}.md`

### Step 6: Review with Developer

Present the generated flows to the developer:
- Show file list and summaries
- Highlight key decisions made during generation
- Offer review options: **[A]ccept / [F]eedback / [R]eject**
- Iterate on feedback until accepted

### Step 7: Install

Init is **not just rendering output** — it must persist enough of ai-flow-anything into the project that the *next* AI session can re-enter the workflow without you re-explaining anything. Generate-and-forget breaks the system: the platform wrapper says "Read `instructions.md` before acting on any ai-flow-anything intent," and that file must actually exist locally for that to work.

Persist the following into the user's project (in roughly this order):

#### 7.1 Platform wrapper (per Step 0 host tool)

For each tool identified in Step 0, install the matching wrapper(s) to its canonical path. See `platforms/{tool}/README.md` for the exact install command. The summary:

| Tool | Source | Destination |
|---|---|---|
| OpenCode | `platforms/opencode/SKILL.md` (orchestrator) **plus** all seven flow skill bodies in `platforms/opencode/flow-skills/{flow}/SKILL.md` **plus** all seven command files in `platforms/opencode/commands/{name}.md` | `.opencode/skills/ai-flow-anything/SKILL.md`, `.opencode/skills/{design,implement,orchestrate,pr,test,deploy,docs}-flow/SKILL.md`, AND `.opencode/commands/{design-flow,implement-flow,orchestrate-flow,test-flow,pr-flow,deploy-flow,docs-flow}.md` (8 skills + 7 commands = 15 files) |
| Claude Code | `platforms/claude/SKILL.md` | `.claude/skills/ai-flow-anything/SKILL.md` |
| Cursor | `platforms/cursor/ai-flow-anything.mdc` | `.cursor/rules/ai-flow-anything.mdc` |
| GitHub Copilot | `platforms/github-copilot/copilot-instructions.md` | `.github/copilot-instructions.md` (append, do not overwrite if file already exists) |
| Kimi-Code CLI | `platforms/kimi-code/AGENTS.md` | `AGENTS.md` at repo root (append, do not overwrite — see notes in `platforms/kimi-code/README.md`) |

**Why OpenCode installs eight skills plus seven commands:** OpenCode has two concepts — *skills* (`.opencode/skills/{name}/SKILL.md`, agent decides via tool call when to load) and *commands* (`.opencode/commands/{name}.md`, user types `/{name}` to invoke directly). They're complementary, not redundant. Skills give the agent strong description-driven matching for natural-language requests; commands give the developer a one-keystroke way to start a flow without going through the `/skills` picker. Installing only skills leaves the developer with a two-click invocation (`/skills` → pick); installing both gets parity with Claude Code's `/design-flow <task>` slash UX. Each command file (~15–25 lines) is a prompt template that loads the matching flow skill and passes `$ARGUMENTS` as the task/task-flow.

Prefer **symlinks** over copies if the OS supports them, so `git pull` in `.ai-workflow/` propagates wrapper updates automatically. Fall back to copy when symlinks aren't available.

#### 7.2 Universal entry point + universal/ + detected profile

The wrapper points at `instructions.md`, and `instructions.md` references `universal/rules.md` etc. — these files must be reachable from the project. Two valid layouts:

**Layout A — project-level install (self-contained, recommended for most teams):**

Clone or copy the ai-flow-anything core into `.ai-workflow/` so the project owns its own copy:

```
.ai-workflow/
├── instructions.md          ← from ai-flow-anything root
├── universal/               ← copy of universal/ from ai-flow-anything
├── profiles/
│   ├── generic/             ← always installed (canonical-flow fallback)
│   └── {detected}/          ← the detected profile (if not generic)
├── flows/                   ← rendered output (Step 5)
└── rules.md                 ← project-specific overrides (Step 7.4)
```

**Why install `profiles/generic/` even when the detected profile is not generic.** Slim profile skeletons (e.g. `profiles/godot-game/skeletons/implement-flow.md`) use the *defer pattern* — they contain a short "follows the canonical structure from `profiles/generic/skeletons/{flow}.md`" header and only add tech-specific overlay content. The defer reference is a real cross-reference the AI is expected to resolve at runtime. If `profiles/generic/` is not installed, the defer references dangle and the rendered slim flow is missing the bulk of its content (sub-tasks, gate definitions, sub-agent mode, etc.). Always install both.

**Layout B — global install + symlink (only viable when the ai-flow-anything clone is project-private):**

ai-flow-anything lives once at e.g. `~/tools/ai-flow-anything/`. The project's `.ai-workflow/` symlinks to it:

```bash
ln -s ~/tools/ai-flow-anything .ai-workflow
# then write .ai-workflow/flows/*.md and .ai-workflow/rules.md
```

⚠️ **Caveat:** the rendered `.ai-workflow/flows/` and `.ai-workflow/rules.md` are written *into* the global ai-flow-anything clone (since `.ai-workflow/` resolves to that clone). They become uncommitted additions to that clone's working tree, which collides with `git pull` updates and conflates project-specific output with ai-flow-anything core.

This is only safe if **one project owns the ai-flow-anything clone** (e.g. `~/projects/myapp/.ai-flow-anything/` symlinked from `~/projects/myapp/.ai-workflow/`). It does *not* work when multiple projects share `~/tools/ai-flow-anything/` — they'd all write their flows into the same place.

For shared global clones, prefer **Layout A** instead — copy the relevant subset into the project, accept the drift cost, and `git pull` the global clone separately when wrappers/rules update.

If unsure, use Layout A. It survives moves, doesn't depend on a global path, and keeps project output cleanly separated from ai-flow-anything core.

#### 7.3 Rendered flows

Write the flows generated in Step 5 to `.ai-workflow/flows/{flow-name}.md`. These are project-specific renders of the profile skeletons.

#### 7.4 Project-specific rules

If the discovery step turned up project-specific rules that don't belong in the universal or profile layer (e.g. "exactly 5 autoloads" for a specific game), write them to `.ai-workflow/rules.md` with severity tags. These layer on top of `universal/rules.md` and the profile's `rules.md` at runtime.

#### 7.5 Knowledge base scaffolding

Create the initial knowledge base structure per `universal/knowledge-base-spec.md`:

```
flow-storage/
├── project/
│   ├── ARCHITECTURE.md
│   ├── CONVENTIONS.md
│   ├── PATTERNS.md
│   └── DECISIONS.md
├── team/
│   ├── onboarding.md
│   ├── workflows.md
│   └── glossary.md
└── tasks/                ← empty, populated by design-flow
```

Populate `project/*.md` from discovery findings (architecture, naming conventions, ADRs surfaced from existing docs). Leave `tasks/` empty.

**Re-init:** If `flow-storage/project/*.md` files already exist (prior init), do NOT overwrite them blindly. Compare against discovery findings and only add new information that doesn't contradict what's already present. The KB accumulates — re-init should enrich, not replace.

#### 7.6 Project-level directive (AGENTS.md / CLAUDE.md augmentation)

**Why this step exists:** Most AI tools read project-root files (`AGENTS.md`, `CLAUDE.md`) as default session context, *separately* from the platform skill / wrapper file they may also discover. A platform wrapper alone is not enough — for tools where skills are pull-not-push (OpenCode invokes skills via a `skill` tool call the agent decides whether to make), the agent has no project-level pressure to defer to ai-flow-anything and may just improvise on natural-language requests like "design task X". The fix is to put a directive in the file the tool *always* reads.

**Action:** If `AGENTS.md` or `CLAUDE.md` exists at the project root (or both), append a ai-flow-anything directive section to each. Skip this step for the Kimi-Code CLI install path — there `AGENTS.md` *is* the ai-flow-anything wrapper, the directive is implicit. If neither file exists and the host tool is one that reads them (OpenCode reads `AGENTS.md`, Claude Code reads `CLAUDE.md`), create the file with just the directive.

**Idempotency:** Use a sentinel comment so re-runs don't duplicate. Wrap the directive in:

```markdown
<!-- ai-flow-anything-directive-start -->
... directive content ...
<!-- ai-flow-anything-directive-end -->
```

On re-init, replace the content between the sentinels rather than appending again.

**Directive content** (paste this verbatim, then customize the rendered flow table if the project added custom flows):

```markdown
<!-- ai-flow-anything-directive-start -->
## ai-flow-anything Workflow (Mandatory)

This project uses **ai-flow-anything** for task development. Before writing any code for a task, you MUST run the design-flow and obtain a signed-off `flow-storage/tasks/{task-name}/design/task-design.md`.

When the developer's request matches one of the intents below, load and follow the corresponding flow from `.ai-workflow/flows/`. On platforms where flows are exposed as a skill/tool (OpenCode `skill` tool, Claude Code skill, etc.), invoke that mechanism to load ai-flow-anything before acting.

| Developer says | Flow |
|---|---|
| "design task X" / "let's design X" | `.ai-workflow/flows/design-flow.md` |
| "implement X" / "next task flow" | `.ai-workflow/flows/implement-flow.md` |
| "orchestrate X" / "build everything for X" | `.ai-workflow/flows/orchestrate-flow.md` |
| "test X" / "add tests" | `.ai-workflow/flows/test-flow.md` |
| "validate X for PR" / "PR check" | `.ai-workflow/flows/pr-flow.md` |
| "deploy X" / "build" | `.ai-workflow/flows/deploy-flow.md` |
| "update docs" / "docs for X" | `.ai-workflow/flows/docs-flow.md` |

**Hard rules — do not violate:**

- No task code without a signed-off `task-design.md`. If the developer asks to implement a task with no prior design, run **design-flow first** even if they didn't ask for it.
- Every flow phase ends with `[A]ccept / [F]eedback / [R]eject`. Never auto-accept. Present the gate to the developer.
- Universal rules in `.ai-workflow/universal/rules.md` are non-negotiable. Profile rules and `.ai-workflow/rules.md` layer on top.

For the full workflow contract, read `.ai-workflow/instructions.md`.
<!-- ai-flow-anything-directive-end -->
```

**Where this directive lands per host tool:**

| Host tool (Step 0) | Directive target | Notes |
|---|---|---|
| OpenCode CLI | `AGENTS.md` | OpenCode reads `AGENTS.md` as default context. Append/update the sentinel block. |
| Claude Code | `CLAUDE.md` | Append/update the sentinel block. The Claude SKILL.md command-alias table already gives the agent slash invocations, but the directive in `CLAUDE.md` ensures the agent defers even when the developer uses natural language. |
| Cursor | `AGENTS.md` *if it exists* | Cursor's `.mdc` rules drive primary behavior; the AGENTS.md directive is belt-and-suspenders for cross-tool consistency. |
| GitHub Copilot | n/a | The Copilot wrapper *is* `.github/copilot-instructions.md` and is always loaded; no separate AGENTS.md needed. |
| Kimi-Code CLI | n/a | Skip — `AGENTS.md` is the wrapper itself. |

If multiple host tools were chosen in Step 0, write the directive to every applicable target.

---

### Step 8: Verify Install

Before declaring init complete, confirm every artifact landed. The AI must be able to answer "yes" to all of these:

- [ ] **Wrapper(s) installed** — the file(s) at the destination(s) from the table in Step 7.1 exist and (if symlinks) resolve to the ai-flow-anything source. For OpenCode specifically, verify all **eight** skills are present: `.opencode/skills/{ai-flow-anything,design-flow,implement-flow,orchestrate-flow,pr-flow,test-flow,deploy-flow,docs-flow}/SKILL.md`, AND all **seven** commands: `.opencode/commands/{design-flow,implement-flow,orchestrate-flow,test-flow,pr-flow,deploy-flow,docs-flow}.md`. The skills picker should show 8 entries; typing `/` should suggest the 7 commands. Either missing means the install is incomplete.
- [ ] **`instructions.md` reachable** — either at `.ai-workflow/instructions.md` (Layout A) or via a symlink resolving to ai-flow-anything (Layout B). The wrapper's "Read `instructions.md` before acting" sentence must not be a dangling reference.
- [ ] **`universal/rules.md` reachable** — at `.ai-workflow/universal/rules.md` or via the symlink.
- [ ] **Detected profile reachable** — at `.ai-workflow/profiles/{detected}/` or via the symlink. At minimum the four files `README.md`, `discovery.md`, `rules.md`, and `skeletons/` must be present.
- [ ] **Generic profile reachable as fallback** — at `.ai-workflow/profiles/generic/`. Always required because slim profile skeletons defer to `profiles/generic/skeletons/{flow}.md` for canonical content. If detected profile *is* generic, this is the same install as the line above and only one copy is needed.
- [ ] **Flows rendered** — `.ai-workflow/flows/*.md` exists, no leaked `{slot}` placeholders (Rule 12), phase gates `[A]/[F]/[R]` intact.
- [ ] **Knowledge base scaffold** — `flow-storage/project/`, `flow-storage/team/`, `flow-storage/tasks/` exist with the expected files.
- [ ] **Project directive present** — for OpenCode, Claude Code, or Cursor host tools, the matching root file (`AGENTS.md` and/or `CLAUDE.md`) contains the ai-flow-anything directive between `<!-- ai-flow-anything-directive-start -->` and `<!-- ai-flow-anything-directive-end -->` sentinels. Skip for Copilot and Kimi-Code (their wrappers serve this role). Without this, the agent has no project-level pressure to defer to flows on natural-language requests and may improvise task work — making the install functionally inert even though all files are in place.
- [ ] **Smoke test** — re-read the wrapper file you just installed and confirm the path it references (`instructions.md`) resolves. If it doesn't, the install is broken.

If any check fails, fix it before reporting "ai-flow-anything initialized" to the developer. Do not present the install as successful when the next session won't be able to load the workflow.

---

## File Reading Order

When acting on any ai-flow-anything intent, read files in this order. Paths are relative to the ai-flow-anything root, which after install is reachable in the user's project via either `.ai-workflow/` (Layout A) or a symlink at `.ai-workflow/` (Layout B). For a fresh init *before* install, read directly from the ai-flow-anything checkout the wrapper points at.

```
1. universal/rules.md                    (always)
2. universal/workflow-structure.md       (always — phase architecture)
3. profiles/{detected}/README.md         (after detection)
4. profiles/{detected}/discovery.md      (after detection)
5. profiles/{detected}/rules.md          (after detection)
6. profiles/{detected}/skeletons/*.md    (during generation)
7. universal/diagram-standards.md        (when generating diagrams)
8. universal/knowledge-base-spec.md      (when creating KB)
9. .ai-workflow/rules.md                 (if present — project-specific overrides, layered last)
```

---

## Project Detection Reference

### Unity Game
**Indicators:**
- `Assets/` directory exists
- `*.cs` files present
- `ProjectSettings/ProjectSettings.asset` exists
- `Packages/manifest.json` exists

**Confidence:**
- 4/4 matches: "This is definitely a Unity project"
- 3/4 matches: "Likely Unity — ask developer to confirm"

### Godot Game
**Indicators:**
- `project.godot` exists
- `*.gd` files present
- `scene/` or `scenes/` directory

### Web Frontend
**Indicators:**
- `package.json` exists
- `src/` directory with `.tsx`, `.jsx`, `.vue`, or `.svelte` files
- `vite.config.*`, `webpack.config.*`, or `next.config.*` exists

### Backend API
**Indicators:**
- `pyproject.toml`, `go.mod`, `Cargo.toml`, `package.json` with server frameworks
- `src/` or `app/` with API route patterns
- `Dockerfile` or `docker-compose.yml`

### Mobile App
**Indicators:**
- Flutter: `pubspec.yaml`, `lib/`, `ios/`, `android/`
- React Native: `package.json`, `ios/`, `android/`, `metro.config.js`

### Generic
**Indicators:**
- Any project with files
- Use as fallback when no other profile matches

---

## Custom Profile Creation

If no profile matches, ask the developer:

> "No built-in profile matches your project. Would you like to:
> 1. Use the **generic** profile (minimal, works for anything)
> 2. **Create a custom profile** from scratch (I'll guide you)
> 3. **Extend an existing profile** (closest match: {best-guess})"

If they choose "Create custom," follow `docs/how-to-create-profile.md`.

---

## Knowledge Base Integration

After flow generation, the knowledge base structure is:

```
flow-storage/
├── project/
│   ├── ARCHITECTURE.md       (created during init)
│   ├── CONVENTIONS.md        (created during init)
│   ├── PATTERNS.md           (populated by flows)
│   └── DECISIONS.md          (ADRs from design phase)
├── team/
│   ├── onboarding.md         (team-specific)
│   ├── workflows.md          (team-specific)
│   └── glossary.md           (domain terms)
└── tasks/
    └── {task-name}/
        ├── design/                                    # design-flow output
        │   ├── task-design.md
        │   ├── task-technical-design.md
        │   ├── task-edge-cases.md
        │   └── diagrams/
        ├── implement/                                 # implement-flow output
        │   └── flow-plan/
        │       └── task-flow-{task-flow-name}.md
        ├── test/                                      # test-flow output
        ├── pr/                                        # pr-flow output
        │   └── feedback/
        ├── deploy/                                    # deploy-flow output
        └── docs/                                      # docs-flow output
            └── lessons-learned.md
```

---

## Flow Execution

When the developer asks to run a flow against a task (for example, "design task my-task" or "implement the next task flow for my-task"):

1. Read the corresponding flow file in `.ai-workflow/flows/` (e.g., `design-flow.md`)
2. Follow its instructions step by step
3. Load project context from the knowledge base
4. Execute phases, producing the artifacts the flow specifies
5. Present for developer review at each phase gate

The same pattern applies to all flow types — design, implement, pr, test, deploy, docs — and to any custom flows the project has added.

---

## Customization Points

Users can customize without modifying ai-flow-anything core:

- **`.ai-workflow/rules.md`** — Project-specific rules (merged with universal + profile)
- **`.ai-workflow/flows/*.md`** — Custom flows or overrides
- **`flow-storage/project/CONVENTIONS.md`** — Team coding standards
- **`flow-storage/project/PATTERNS.md`** — Reusable patterns discovered by flows

---

## Important Notes

- **Never skip developer review.** Every phase ends with [A]ccept / [F]eedback / [R]eject.
- **Never auto-accept.** The AI makes suggestions, the developer decides.
- **Unlimited iterations.** No maximum review rounds. Quality over speed.
- **Living documentation.** Docs are updated at every step, never archived.
- **Diagrams are mandatory.** Minimum class diagram + package diagram per task.
- **Knowledge capture.** After every task, update lessons-learned and patterns catalog.
