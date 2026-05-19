# ai-flow-anything

> **Universal workflow generator for AI-assisted development.**  
> Markdown-native. Zero installation. Works with any project type.

---

## What is ai-flow-anything?

ai-flow-anything generates tailored AI-assisted development workflows for your project. It interviews your codebase, asks you about your goals, and produces a complete set of flows that guide AI assistants through your development process.

**No code. No DSL. No CLI.** Just markdown files that Claude, Cursor, or any AI assistant reads and follows.

---

## Philosophy

- **AI is the engine.** All instructions are prose in markdown. The AI reads, interprets, and acts.
- **Documentation-first.** Every task starts with design docs and diagrams before code.
- **Living knowledge base.** Documentation evolves with the project, capturing patterns and decisions.
- **Universal by default.** Works for games, web apps, APIs, mobile apps — anything.
- **Customizable by editing markdown.** Add phases, rules, or entirely new flow types by editing files.

---

## Quick Start

### 1. Clone into your project

```bash
cd your-project
git clone https://github.com/yourusername/ai-flow-anything.git .ai-workflow
```

### 2. Activate the wrapper for your AI platform

Pick the platform you use and follow the install instructions in its directory:

| Platform | Install guide |
|----------|---------------|
| Claude Code | [`platforms/claude/README.md`](platforms/claude/README.md) |
| Cursor | [`platforms/cursor/README.md`](platforms/cursor/README.md) |
| GitHub Copilot | [`platforms/github-copilot/README.md`](platforms/github-copilot/README.md) |
| OpenCode | [`platforms/opencode/README.md`](platforms/opencode/README.md) |
| Kimi Code CLI | [`platforms/kimi-code/README.md`](platforms/kimi-code/README.md) |

Each wrapper is a thin file that points the AI at the platform-neutral [`instructions.md`](instructions.md). All workflow logic is the same; only the activation mechanism differs per tool.

### 3. Initialize ai-flow-anything

Ask your AI to initialize ai-flow-anything for the project. Each platform's install guide shows the exact invocation — slash command on platforms that support them (like Claude Code), or natural language anywhere ("initialize ai-flow-anything for this project"). The AI will:

1. **Detect** your project type (Unity, Godot, React, FastAPI, etc.)
2. **Ask** you about project-specific decisions
3. **Generate** flows tailored to your project
4. **Install** them to `.ai-workflow/flows/`

### 4. Use your flows

Ask the AI to run a flow against a task. Examples (use whatever invocation style your platform supports — slash command on Claude Code, natural language anywhere):

- "Design a task called my-task" — runs the design flow
- "Implement the next task flow for my-task" — runs the implement flow
- "Validate my-task for PR" — runs the PR flow

---

## What You Get

After initializing ai-flow-anything, your project has:

```
.ai-workflow/
├── flows/
│   ├── design-flow.md              # Design tasks with diagrams
│   ├── implement-flow.md           # Implement task flows with validation
│   ├── free-flow.md                # Quick fixes, tweaks, small refactors
│   ├── parallel-implement-flow.md  # Parallel implementation of all task flows
│   ├── pr-flow.md                  # PR lifecycle management
│   ├── test-flow.md                # Test planning & execution
│   ├── deploy-flow.md              # Deployment & release
│   └── docs-flow.md                # Documentation maintenance
│
└── knowledge-base/
    ├── project/              # Architecture, conventions, patterns
    └── team/                 # Onboarding, workflows, glossary
```

Plus a per-task knowledge base that grows with every task:

```
flow-storage/tasks/my-task/
├── design/                                       # design-flow output
│   ├── task-design.md                            # Design document (immutable after sign-off)
│   ├── task-technical-design.md                  # Technical design (evolves)
│   ├── task-edge-cases.md                        # Edge cases & decisions
│   └── diagrams/
│       ├── class-diagram.puml
│       ├── package-diagram.puml
│       └── sequence-diagram.puml
├── implement/                                    # implement-flow output
│   └── flow-plan/
│       ├── task-flow-01-user-session-store.md    # AI proposes descriptive names
│       ├── task-flow-02-auth-service.md          # Variable number (2-10+) based on complexity
│       ├── task-flow-03-login-form-component.md
│       └── task-flow-04-tests.md                 # Developer approves/modifies
├── test/                                         # test-flow output
├── pr/                                           # pr-flow output
│   └── feedback/                                 # PR feedback records
├── deploy/                                       # deploy-flow output
└── docs/                                         # docs-flow output
    └── lessons-learned.md                        # Post-merge summary
```

---

## Project Types Supported

| Profile | Description | Detection |
|---------|-------------|-----------|
| **generic** | Minimal fallback — works for anything | Any project |
| **unity-game** | Unity/C# games | `Assets/`, `*.cs`, `ProjectSettings/` |
| **godot-game** | Godot/GDScript games | `project.godot`, `*.gd` |
| **web-frontend** | React/Vue/Angular/Svelte | `package.json`, `src/` |
| **backend-api** | FastAPI/Django/Express/Go | `pyproject.toml`, `go.mod`, `Cargo.toml` |
| **mobile-app** | Flutter/React Native | `pubspec.yaml`, `ios/`/`android/` |

---

## How It Works

```
Developer asks to initialize ai-flow-anything.

AI reads:
  instructions.md                → "I am a workflow generator"
  universal/rules.md              → "These rules apply to all projects"
  profiles/*/README.md            → "Which profile matches this project?"
  
AI detects project type:
  → Scans files, matches against profile detection hints
  → If ambiguous: asks developer
  → If no match: offers to create custom profile

AI loads profile:
  profiles/{detected}/README.md   → "This is a Unity game"
  profiles/{detected}/discovery.md → "Explore the codebase like this..."
  profiles/{detected}/rules.md    → "Follow these Unity-specific rules"
  profiles/{detected}/skeletons/  → "Fill these templates with discovered details"

AI executes discovery:
  → Follows discovery.md instructions
  → Reads files, searches patterns, asks questions
  → Fills slot catalog with project specifics

AI generates flows:
  → Loads skeletons
  → Fills project-specific slots (class names, namespaces, patterns)
  → Applies universal + profile rules
  → Writes to .ai-workflow/flows/

AI presents for review:
  → Shows generated files
  → Developer: [A]ccept / [F]eedback / [R]eject
  → Iterates until accepted

Developer uses flows by asking:
  → "Design a task called my-task"
  → "Implement the next task flow for my-task"
  → "Validate my-task for PR"
```

---

## Customization

### Add a custom rule

Edit `.ai-workflow/rules.md` in your project:

```markdown
## Custom Rule: No console.log in production

Severity: error
Check: No `console.log` statements in `src/` unless inside `if (DEBUG)`.
```

### Add a custom flow

Create `.ai-workflow/flows/my-flow.md`:

```markdown
# Flow: Security Audit

## Purpose
Run security checks on new tasks.

## Phases
1. Scan for SQL injection risks
2. Check auth middleware coverage
3. Validate input sanitization
4. Report findings
```

### Create a new profile

See [docs/how-to-create-profile.md](docs/how-to-create-profile.md).

---

## Documentation

- [Quick Start](docs/quick-start.md) — 5-minute setup
- [How to Create a Profile](docs/how-to-create-profile.md) — Add support for your tech stack
- [How to Write Rules](docs/how-to-write-rules.md) — Define project conventions
- [How to Add a Flow](docs/how-to-add-flow.md) — Extend the workflow
- [How to Customize](docs/how-to-customize.md) — Project-level overrides
- [Architecture Explained](docs/architecture-explained.md) — Why markdown-native?

---

## Contributing

Profiles are community-contributed. To add a new profile:

1. Copy `profiles/generic/` as a starting point
2. Edit `README.md`, `discovery.md`, `rules.md`
3. Customize skeletons for your tech stack
4. Submit a pull request

---

## License

MIT
