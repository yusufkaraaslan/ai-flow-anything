# Kimi Code CLI wrapper

Install ai-flow-anything for use with [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) (Moonshot AI's terminal coding agent).

## ⚠️ Important caveat: manual invocation required

**Kimi Code CLI does not currently auto-load `AGENTS.md` at session start** ([upstream issue #850](https://github.com/MoonshotAI/kimi-cli/issues/850)). After installing the wrapper, you need to reference `AGENTS.md` in your first prompt of each session, for example:

```
@AGENTS.md  please initialize ai-flow-anything for this project
```

or

```
Please read AGENTS.md, then design a task called auth.
```

Once Kimi adds auto-loading (the task is requested but not yet shipped as of this writing), the wrapper will activate without manual invocation — no changes to your install needed.

## Install

From your project root:

```bash
git clone https://github.com/yusufkaraaslan/ai-flow-anything.git .ai-workflow
ln -s .ai-workflow/platforms/kimi-code/AGENTS.md AGENTS.md
```

The symlink lets `git pull` in `.ai-workflow/` update the wrapper automatically. If your environment doesn't support symlinks, copy instead:

```bash
cp .ai-workflow/platforms/kimi-code/AGENTS.md AGENTS.md
```

## Already have an `AGENTS.md`?

`AGENTS.md` is a cross-tool convention used by Kimi Code, OpenAI Codex, and several other emerging CLI agents. Append the ai-flow-anything wrapper's content to your existing file rather than overwriting it. Put ai-flow-anything last so it doesn't dominate the steering context:

```bash
echo "" >> AGENTS.md
cat .ai-workflow/platforms/kimi-code/AGENTS.md >> AGENTS.md
```

## How it works

Once Kimi Code reads `AGENTS.md` (either auto-loaded in a future release, or manually referenced in your first prompt), it learns:

1. ai-flow-anything is installed in this repo
2. Which developer intents should trigger the workflow
3. Where to find the full logic (`instructions.md`)

When the developer expresses a ai-flow-anything intent, Kimi reads `instructions.md` (and the universal/profile/skeleton files it references) and executes the workflow.

## Update

```bash
cd .ai-workflow && git pull
```

The symlink picks up any wrapper changes automatically.

## Uninstall

```bash
rm AGENTS.md
rm -rf .ai-workflow
```

If you appended ai-flow-anything to an existing `AGENTS.md`, manually delete the ai-flow-anything section instead of removing the whole file.
