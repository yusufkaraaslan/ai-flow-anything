# OpenCode wrapper

Install ai-flow-anything for use with [OpenCode](https://opencode.ai).

## What gets installed

**8 skills + 7 commands = 15 files.** The skills are for agent auto-invocation (description-driven); the commands are for user-typed slash invocation. They complement each other.

### Skills (8)

| Skill | Source | Role |
|---|---|---|
| `ai-flow-anything` | `platforms/opencode/SKILL.md` | Orchestrator — init, status, knowledge-base search, ambiguous requests |
| `design-flow` | `platforms/opencode/flow-skills/design-flow/SKILL.md` | Phase: design (task-design.md, task-technical-design.md, diagrams, task flows) |
| `implement-flow` | `platforms/opencode/flow-skills/implement-flow/SKILL.md` | Phase: implement one task flow (also sub-agent mode for orchestrator) |
| `orchestrate-flow` | `platforms/opencode/flow-skills/orchestrate-flow/SKILL.md` | Phase: parallel orchestration of all task flows via worktrees + subagents |
| `pr-flow` | `platforms/opencode/flow-skills/pr-flow/SKILL.md` | Phase: PR validation, feedback, post-merge capture |
| `test-flow` | `platforms/opencode/flow-skills/test-flow/SKILL.md` | Phase: tests |
| `deploy-flow` | `platforms/opencode/flow-skills/deploy-flow/SKILL.md` | Phase: build/export/release |
| `docs-flow` | `platforms/opencode/flow-skills/docs-flow/SKILL.md` | Phase: knowledge-base maintenance |

Each flow skill has an imperative description naming its trigger phrases. The agent uses these descriptions to decide whether to invoke `skill({ name })` when the user makes a natural-language request like "design task X."

### Commands (7)

| Command | Source | Invocation |
|---|---|---|
| `/design-flow <task>` | `platforms/opencode/commands/design-flow.md` | Run design-flow for a task |
| `/implement-flow <task> [task-flow]` | `platforms/opencode/commands/implement-flow.md` | Implement one task flow from a designed task |
| `/orchestrate-flow <task>` | `platforms/opencode/commands/orchestrate-flow.md` | Orchestrate all task flows in parallel |
| `/test-flow [task]` | `platforms/opencode/commands/test-flow.md` | Run or add tests |
| `/pr-flow <task> [mode]` | `platforms/opencode/commands/pr-flow.md` | Pre-PR validation, feedback, or capture |
| `/deploy-flow [task]` | `platforms/opencode/commands/deploy-flow.md` | Build verification + smoke test |
| `/docs-flow [scope]` | `platforms/opencode/commands/docs-flow.md` | Audit/update knowledge base |

Each command is a prompt template that loads the matching flow skill and passes `$ARGUMENTS` as the task/task-flow. Without these, the developer has to either rely on natural-language matching to the skills (works but unreliable) or use `/skills → pick` (two clicks). With commands, `/design-flow board-system` is one keystroke.

## Install

OpenCode supports both project-level and global skills. Pick the install path that matches your scope.

### Project-level (recommended)

Use this when ai-flow-anything should be available only inside one repository.

From your project root:

```bash
git clone https://github.com/yourusername/ai-flow-anything.git .ai-workflow

# Install 7 skills + 6 commands as symlinks (preferred):
for skill in ai-flow-anything design-flow implement-flow orchestrate-flow pr-flow test-flow deploy-flow docs-flow; do
  mkdir -p ".opencode/skills/$skill"
  if [ "$skill" = "ai-flow-anything" ]; then
    ln -s "../../../.ai-workflow/platforms/opencode/SKILL.md" ".opencode/skills/$skill/SKILL.md"
  else
    ln -s "../../../.ai-workflow/platforms/opencode/flow-skills/$skill/SKILL.md" ".opencode/skills/$skill/SKILL.md"
  fi
done

mkdir -p ".opencode/commands"
for cmd in design implement orchestrate test pr deploy docs; do
  ln -s "../../.ai-workflow/platforms/opencode/commands/$cmd.md" ".opencode/commands/$cmd.md"
done
```

Symlinks let `git pull` in `.ai-workflow/` update every wrapper automatically. If your environment doesn't support symlinks, replace `ln -s` with `cp`:

```bash
for skill in ai-flow-anything design-flow implement-flow orchestrate-flow pr-flow test-flow deploy-flow docs-flow; do
  mkdir -p ".opencode/skills/$skill"
  if [ "$skill" = "ai-flow-anything" ]; then
    cp ".ai-workflow/platforms/opencode/SKILL.md" ".opencode/skills/$skill/SKILL.md"
  else
    cp ".ai-workflow/platforms/opencode/flow-skills/$skill/SKILL.md" ".opencode/skills/$skill/SKILL.md"
  fi
done

mkdir -p ".opencode/commands"
for cmd in design implement orchestrate test pr deploy docs; do
  cp ".ai-workflow/platforms/opencode/commands/$cmd.md" ".opencode/commands/$cmd.md"
done
```

**Caveat for copy installs:** when you `git pull` in `.ai-workflow/`, you must re-run the loop above to refresh the copies. OpenCode's server caches skills in memory at startup — after refreshing, restart the server (not just the TUI) for changes to take effect.

### Global

Use this when you want ai-flow-anything available across every project, with the ai-flow-anything core cloned somewhere stable (e.g. `~/tools/ai-flow-anything/`).

```bash
git clone https://github.com/yourusername/ai-flow-anything.git ~/tools/ai-flow-anything
mkdir -p ~/.config/opencode/skills/ai-flow-anything
ln -s ~/tools/ai-flow-anything/platforms/opencode/SKILL.md ~/.config/opencode/skills/ai-flow-anything/SKILL.md
```

You'll still need a per-project `.ai-workflow/` (or equivalent) holding `instructions.md`, `universal/`, and `profiles/` for the AI to read. The simplest pattern is to clone ai-flow-anything once globally and symlink `.ai-workflow` into each project:

```bash
cd your-project
ln -s ~/tools/ai-flow-anything .ai-workflow
```

## How it works

OpenCode reads `SKILL.md` files under skill directories and exposes each as a tool the agent can call. The YAML frontmatter `description` is what the agent sees when deciding whether to invoke the skill — but **invocation is pull, not push**: the agent decides whether to call `skill({ name: "ai-flow-anything" })` to load the body. There's no auto-activation purely from a description match.

This means installing `.opencode/skills/ai-flow-anything/SKILL.md` alone is **not enough** to make ai-flow-anything work. Without project-level pressure, the agent may improvise task work instead of loading the skill. Step 7.6 of `instructions.md` covers this by also writing a ai-flow-anything directive to `AGENTS.md` (which OpenCode reads as default project context every session). When init is done correctly, both artifacts are in place.

ai-flow-anything uses the standard skill frontmatter fields:

- `name: ai-flow-anything` — the skill identifier (becomes tool `skills_ai_flow_anything`)
- `description` — what the agent sees in the tool list; ours is imperative and lists trigger phrases
- `license`, `metadata` — informational

When loaded, the skill points OpenCode at `instructions.md` for the full workflow logic.

OpenCode also reads `.claude/skills/` and `.agents/skills/` for cross-tool compatibility, but the canonical install location for the OpenCode-native wrapper is `.opencode/skills/ai-flow-anything/SKILL.md`.

## Update

```bash
cd .ai-workflow && git pull            # project-level install
cd ~/tools/ai-flow-anything && git pull       # global install
```

The symlink picks up any wrapper changes automatically.

## Uninstall

```bash
# project-level
rm -rf .opencode/skills/ai-flow-anything
rm -rf .ai-workflow

# global
rm -rf ~/.config/opencode/skills/ai-flow-anything
# (keep ~/tools/ai-flow-anything if other projects symlink to it)
```
