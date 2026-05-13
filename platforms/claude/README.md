# Claude Code wrapper

Install ai-flow-anything for use with [Claude Code](https://claude.com/claude-code).

## Install

From your project root:

```bash
git clone https://github.com/yourusername/ai-flow-anything.git .ai-workflow

# Symlinks (preferred — `git pull` in .ai-workflow/ then propagates updates automatically):

# 1. Master skill
mkdir -p .claude/skills/ai-flow-anything
ln -s ../../../.ai-workflow/platforms/claude/SKILL.md .claude/skills/ai-flow-anything/SKILL.md

# 2. Per-flow skill: orchestrate-flow
mkdir -p .claude/skills/orchestrate-flow
ln -s ../../../.ai-workflow/platforms/claude/flow-skills/orchestrate-flow/SKILL.md .claude/skills/orchestrate-flow/SKILL.md

# 3. Custom subagent for orchestrate-flow's parallel implementers
mkdir -p .claude/agents
ln -s ../../.ai-workflow/platforms/claude/agents/orchestrate-implementer.md .claude/agents/orchestrate-implementer.md

# 4. Slash command for orchestrate-flow
mkdir -p .claude/commands
ln -s ../../.ai-workflow/platforms/claude/commands/orchestrate-flow.md .claude/commands/orchestrate-flow.md
```

If your environment doesn't support symlinks, copy each file to the same destination.

**What gets installed and why:**

| File | Why it's needed |
|---|---|
| `.claude/skills/ai-flow-anything/SKILL.md` | Master orchestrator skill — matches initialization, status, and ambiguous-intent requests |
| `.claude/skills/orchestrate-flow/SKILL.md` | Per-flow skill — strong description match for "orchestrate {task}" / "build everything for {task}" |
| `.claude/agents/orchestrate-implementer.md` | Custom subagent with `isolation: worktree` — auto-creates per-subagent worktrees, no manual `git worktree add` needed |
| `.claude/commands/orchestrate-flow.md` | Slash command `/orchestrate-flow <task>` for one-keystroke invocation |

Only `orchestrate-flow` currently ships with its own per-flow skill, subagent, and command — the other flows (design, implement, pr, test, deploy, docs) are run via the master skill and the natural-language intents listed in its command alias table.

## How it works

Claude Code auto-discovers four kinds of plugin files under `.claude/`:

- **Skills** (`.claude/skills/{name}/SKILL.md`) — loaded on demand when the YAML `description` matches the developer's request.
- **Agents** (`.claude/agents/{name}.md`) — custom subagent types invocable via the Agent tool's `subagent_type` parameter; `isolation: worktree` in their frontmatter auto-creates a per-invocation git worktree.
- **Commands** (`.claude/commands/{name}.md`) — slash commands the developer types directly (e.g. `/orchestrate-flow user-auth`).
- The master skill's command alias table maps the legacy `/ai-flow-anything …` and `/{flow}-flow <task>` invocations to ai-flow-anything intents.

All of them point at `instructions.md` (in `.ai-workflow/`) for the full workflow logic — the wrappers are intentionally thin so the same core logic works on every platform.

**Orchestrate-flow specifically** uses all four wrapper types: the master skill (for ambiguous requests), the per-flow skill (for strong "orchestrate X" matches), the `orchestrate-implementer` subagent (for parallel execution with auto-worktrees), and the `/orchestrate-flow` slash command (for direct invocation). See [`flow-skills/orchestrate-flow/SKILL.md`](flow-skills/orchestrate-flow/SKILL.md) for the parallel-subagent dispatch pattern.

## Update

```bash
cd .ai-workflow && git pull
```

The symlink picks up any wrapper changes automatically.

## Uninstall

```bash
rm -rf .ai-workflow \
  .claude/skills/ai-flow-anything \
  .claude/skills/orchestrate-flow \
  .claude/agents/orchestrate-implementer.md \
  .claude/commands/orchestrate-flow.md
```

This removes ai-flow-anything but leaves your project's own `flow-storage/` (your knowledge base) untouched. If you ran orchestrate-flow previously, any leftover sub-agent branches (`orchestrate-implementer-*`) can be pruned with `git branch -D` — they're independent of the wrapper install.
