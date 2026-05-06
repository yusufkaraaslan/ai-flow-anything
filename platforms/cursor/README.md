# Cursor wrapper

Install ai-flow-anything for use with [Cursor](https://cursor.com).

## Install

From your project root:

```bash
git clone https://github.com/yourusername/ai-flow-anything.git .ai-workflow
mkdir -p .cursor/rules
ln -s ../../.ai-workflow/platforms/cursor/ai-flow-anything.mdc .cursor/rules/ai-flow-anything.mdc
```

The symlink lets `git pull` in `.ai-workflow/` update the wrapper automatically. If your environment doesn't support symlinks, copy instead:

```bash
cp .ai-workflow/platforms/cursor/ai-flow-anything.mdc .cursor/rules/ai-flow-anything.mdc
```

## How it works

Cursor reads `.cursor/rules/*.mdc` files and uses the YAML frontmatter to decide when each rule applies. ai-flow-anything uses the **Apply Intelligently** rule type:

- `description` — Cursor's agent reads this and activates the rule when the developer's request matches.
- `alwaysApply: false` — the rule is not loaded into every chat; only when relevant.

When activated, the rule points Cursor at `instructions.md` (in the cloned `.ai-workflow/` directory) for the full workflow logic.

### Note on the legacy `.cursorrules` file

Older Cursor installs used a single `.cursorrules` file at the project root. The current convention is one or more `.mdc` files under `.cursor/rules/`. If your project still has a `.cursorrules` file, you can keep it for unrelated guidance — ai-flow-anything only adds `.cursor/rules/ai-flow-anything.mdc`.

## Update

```bash
cd .ai-workflow && git pull
```

The symlink picks up any wrapper changes automatically.

## Uninstall

```bash
rm .cursor/rules/ai-flow-anything.mdc
rm -rf .ai-workflow
```

This removes ai-flow-anything but leaves the project's own `flow-storage/` (your knowledge base) and any other `.cursor/rules/` files untouched.
