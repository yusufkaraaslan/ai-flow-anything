# Claude Code wrapper

Install ai-flow-anything for use with [Claude Code](https://claude.com/claude-code).

## Install

From your project root:

```bash
git clone https://github.com/yourusername/ai-flow-anything.git .ai-workflow

# Symlink (preferred):
mkdir -p .claude/skills/ai-flow-anything
ln -s ../../../.ai-workflow/platforms/claude/SKILL.md .claude/skills/ai-flow-anything/SKILL.md
```

The symlink lets `git pull` in `.ai-workflow/` update the wrapper automatically. If your environment doesn't support symlinks, copy instead:

```bash
mkdir -p .claude/skills/ai-flow-anything
cp .ai-workflow/platforms/claude/SKILL.md .claude/skills/ai-flow-anything/SKILL.md
```

## How it works

Claude Code auto-discovers skills in `.claude/skills/` and reads the YAML frontmatter of each `SKILL.md`. When the developer's request matches the skill's `description`, Claude loads the file. The wrapper then:

1. Translates slash commands (`/ai-flow-anything init`, `/design-flow auth`, …) into ai-flow-anything intents.
2. Points Claude at `instructions.md` (in `.ai-workflow/`) for the full workflow logic.

All workflow logic stays in `instructions.md` — the wrapper is intentionally thin so the same core logic works on every platform.

## Update

```bash
cd .ai-workflow && git pull
```

The symlink picks up any wrapper changes automatically.

## Uninstall

```bash
rm -rf .ai-workflow .claude/skills/ai-flow-anything
```

This removes ai-flow-anything but leaves your project's own `flow-storage/` (your knowledge base) untouched.
