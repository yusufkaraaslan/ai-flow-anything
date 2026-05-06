# Profile: Generic

> **Purpose:** Minimal fallback profile that works for any project type  
> **When to Use:** When no specific profile matches, or for simple projects

---

## Description

The generic profile provides a minimal, adaptable workflow for any project. It makes no assumptions about:
- Programming language
- Framework
- Architecture pattern
- Directory structure

It discovers what it can from the codebase and asks the developer about the rest.

---

## Detection Hints

This profile **always matches** with low confidence (1/5). It serves as the fallback when no other profile detects the project.

**Indicators:**
- Any directory with files
- No strong indicators for Unity, Godot, Web, Backend, or Mobile

**Confidence:** "Low — using generic profile as fallback"

---

## Capabilities

| Flow | Description |
|--------|-------------|
| **design-flow** | Task design with diagrams (class, package, sequence) |
| **implement-flow** | Task-flow-by-task-flow implementation |
| **pr-flow** | PR validation, feedback handling, knowledge capture |
| **test-flow** | Test planning and execution |
| **deploy-flow** | Release and deployment |
| **docs-flow** | Documentation maintenance |

---

## Limitations

- No framework-specific code patterns
- No language-specific syntax examples
- Generic directory structure (`src/` assumed)
- Minimal validation (relies on developer review)

**Recommendation:** If your project uses a supported tech stack, use a specific profile instead. This profile is for:
- Unusual tech stacks
- Polyglot projects
- Prototypes and experiments
- Projects not yet ready for conventions

---

## What Gets Discovered

| Slot | Discovery Method | Fallback |
|------|-----------------|----------|
| project-name | README.md heading or directory name | Ask developer |
| source-directory | Check `src/`, `lib/`, `app/`, `code/` | Ask developer |
| primary-language | File extension analysis | Ask developer |
| architecture-pattern | Directory structure + file search | Ask developer |
| test-directory | Check `test/`, `tests/`, `spec/` | Assume `tests/` |

---

## Customization

The generic profile is designed to be customized. After init:

1. Edit `.ai-workflow/rules.md` to add project-specific conventions
2. Edit flow files in `.ai-workflow/flows/` to add language-specific patterns
3. Add custom slots by editing the discovery process

See [docs/how-to-customize.md](../../docs/how-to-customize.md) for details.

---

## Example Projects

- Python scripts and utilities
- Shell script collections
- Data processing pipelines
- Research prototypes
- Multi-language repositories
- Configuration repositories
