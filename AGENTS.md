# ai-flow-anything repository guide

This repo is a **markdown-only workflow generator** — there is no code, no build system, no test suite, and no CLI. The runtime is the AI itself reading and acting on markdown files.

> **Naming note:** This repo was renamed to **ai-flow-anything**. The local directory may still be `magic-seed/` — that's a stale artifact. All references use the new name.

## Hard constraints

- **Never run `git annex` commands** in this repo (sibling-repo convention).
- **No build/test/lint/typecheck** — this repo has none. Do not attempt to run them.
- **Wrappers stay thin** — Platform files in `platforms/{tool}/` must be ~20–80 lines (the main `SKILL.md` / `AGENTS.md` / `*.mdc`). Per-flow skills in `flow-skills/*/SKILL.md` may run up to ~100 lines when they encode platform-specific mechanics. Verify with `find platforms -name '*.md' | xargs wc -l | sort -n`. If a wrapper grows beyond trigger config + pointer to `instructions.md`, the content belongs in `instructions.md` or `universal/` instead.
- **No slash-command syntax outside `platforms/claude/`** — Only the Claude wrapper may define `/ai-flow-anything` slash aliases. Everywhere else, refer to flows by name in prose.
- **Tool ≠ model** — `platforms/{tool}/` directories are keyed on the AI tool (Claude Code, OpenCode, Cursor, Kimi-Code CLI, GitHub Copilot), not the LLM provider. Never use model names for detection signals or wrapper paths.

## Architecture (four layers, strictly ordered)

```
universal/          → Rules, workflow structure, diagram standards, KB spec
profiles/{type}/    → Per-tech-stack detection, discovery, rules, flow skeletons
platforms/{tool}/   → Thin wrapper: trigger semantics + install path + pointer to instructions.md
docs/               → Project documentation (how-to guides, not runtime logic)
```

**The entry point is `instructions.md`** at the repo root. All wrappers reference it; do not duplicate its logic.

## Versioning

`VERSION` at repo root is semver. Bump it for any release-worthy change (new flow, changed gate cadence, new rule) — Step 7.2 of `instructions.md` copies it into installs and the "update ai-flow-anything" intent compares it to detect updates.

## Editing conventions

### Before modifying anything, read in this order:
1. `instructions.md` — orchestration contract
2. `universal/rules.md` — 18 numbered rules with severity (`error` / `warning` / `info`)
3. `universal/workflow-structure.md` — 9 flow types and phase architecture
4. `CLAUDE.md` at repo root — additional hard rules for this repo

### Rules that are load-bearing (do not soften without discussion):
- **Rule 6: No auto-approval** — Every phase ends with `[A]ccept / [F]eedback / [R]eject`. Never auto-accept.
- **Rule 9: task-design.md immutable after sign-off** — Deviations go in `task-technical-design.md`, never edit a signed-off `task-design.md`.
- **Rule 4: Diagrams mandatory for system-class tasks** — Every system task gets at least class + package diagrams; content-class tasks may skip with a recorded one-line justification.
- **Rule 2: Living documents, no archives** — Update in place. Never create `docs-v2/` or dated copies.

### Validation checklist (manual — no test runner):
1. **No leaked placeholders** — Per Rule 12, no `{[a-z][a-z0-9_-]*}` patterns remain in non-template files. Skeletons and `*-TEMPLATE.md` files are exempt.
2. **Slot names match across profile** — Slot names in `skeletons/*.md` must be defined/discovered in the profile's `discovery.md`.
3. **Phase gates intact** — Every top-level flow phase ends with an explicit `[A]/[F]/[R]` gate or is explicitly marked as auto-executing. Sub-tasks within a phase auto-proceed without intermediate gates.
4. **File reading order updated** — If you add new universal files, update the "File Reading Order" section in `instructions.md`.
5. **Wrapper thinness** — Verify `platforms/{tool}/*` files have not grown beyond ~80 lines.
6. **Init persistence** — When changing the init flow, preserve the contract that init produces *all* artifacts: platform wrapper, universal/profile reachable, rendered flows, knowledge base scaffold, and project directive. Generate-and-forget breaks the system.
7. **Profile dirs are exactly four entries** — Every `profiles/{type}/` contains exactly `README.md`, `discovery.md`, `rules.md`, and `skeletons/`. No literal-name placeholder dirs, no extras. Verify with `find profiles -type d -name '{*}'` — must return empty.
8. **Slim profiles require generic installed alongside** — Non-generic profiles (godot-game, web-frontend, backend-api, mobile-app, unity-game) use the *defer pattern*: their skeleton bodies are short overlays that reference `profiles/generic/skeletons/{flow}.md` for canonical structure. Step 7.2 of `instructions.md` installs both profiles. Don't duplicate canonical content into slim profiles.
9. **Root `AGENTS.md` and `CLAUDE.md` stay in sync** — This file is the condensed mirror of `CLAUDE.md` (and is NOT `platforms/kimi-code/AGENTS.md`, the Kimi wrapper for user projects). Changes to shared guidance go to both files in the same commit.
10. **Counts and flow lists stay in sync across docs** — Flow lists, profile/platform tables, and install-file counts are duplicated across `README.md`, `docs/*.md`, `platforms/{tool}/README.md`, and `instructions.md` Step 8. Adding/renaming/removing a flow, profile, or platform requires sweeping all of them.

## Adding a new profile

1. Copy `profiles/generic/` to `profiles/{new-type}/`
2. Edit the four required files:
   - `README.md` — Detection hints + confidence scoring
   - `discovery.md` — Slot discovery script
   - `rules.md` — Tech-stack-specific conventions
   - `skeletons/*.md` — Nine flow templates (design, implement, free, parallel-implement, pr, test, deploy, docs, kb-sync)
3. Update the profile table in `README.md` and `instructions.md`.

## Adding a new platform wrapper

1. Create `platforms/{new-tool}/` with the tool's native instruction file (YAML frontmatter for OpenCode/Claude, `.mdc` for Cursor, etc.). Complex platforms may also need nested subdirs — see `platforms/opencode/` (commands/ + flow-skills/) and `platforms/claude/` (commands/ + flow-skills/ + agents/) for the pattern.
2. Body must: list trigger intents, point at `instructions.md`, and nothing more.
3. Add `README.md` install guide documenting symlink-vs-copy, append-don't-overwrite, and any platform caveats.
4. Update the platform table in `README.md` and `CLAUDE.md`.

## Common mistakes to avoid

- **Duplicating workflow logic in wrappers** — All detection, discovery, generation, and review-gate logic lives in `instructions.md` and `universal/`.
- **Forgetting to update file reading order** — New universal files must be listed in `instructions.md`.
- **Adding slash commands outside Claude wrapper** — This breaks platform neutrality.
- **Creating archives or versioned doc copies** — Update documents in place per Rule 2.
- **Modifying `platforms/kimi-code/AGENTS.md` thinking it's the repo's AGENTS.md** — That file is a thin wrapper for *consumer projects* that install ai-flow-anything. This root `AGENTS.md` is for working *on* ai-flow-anything itself.
