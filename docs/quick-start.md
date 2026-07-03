# Quick Start

Get ai-flow-anything running in 5 minutes.

---

## Step 1: Clone ai-flow-anything

```bash
cd your-project
git clone https://github.com/yusufkaraaslan/ai-flow-anything.git .ai-workflow
```

Or download and extract to `.ai-workflow/`.

---

## Step 2: Activate the wrapper for your AI platform

Follow the install guide for the platform you use:

- Claude Code → [`platforms/claude/README.md`](../platforms/claude/README.md)
- Cursor → [`platforms/cursor/README.md`](../platforms/cursor/README.md)
- GitHub Copilot → [`platforms/github-copilot/README.md`](../platforms/github-copilot/README.md)
- OpenCode → [`platforms/opencode/README.md`](../platforms/opencode/README.md)
- Kimi Code CLI → [`platforms/kimi-code/README.md`](../platforms/kimi-code/README.md)

The wrapper teaches your AI how to recognize ai-flow-anything intents on your platform.

---

## Step 3: Initialize

Ask your AI to initialize ai-flow-anything for the project (use the platform's invocation style — slash command on platforms that support them, natural language anywhere). The AI will:

1. Detect your project type
2. Ask a few questions
3. Generate flows
4. Install them to `.ai-workflow/flows/`

---

## Step 4: Design Your First Task

Ask your AI: "Design a task called my-task."

The AI will:
1. Load your project context
2. Ask about the task
3. Generate diagrams (class, package)
4. Create task-design.md, task-technical-design.md, task-edge-cases.md
5. Create implementation task flows

**Review and approve each phase.**

---

## Step 5: Implement

Ask your AI: "Implement the next task flow for my-task."

The AI will:
1. Read the first task flow
2. Plan implementation
3. Write code
4. Run validation
5. Present for your review

**Approve each task flow before proceeding to the next.**

---

## Step 6: PR

After all task flows are complete, ask your AI: "Validate my-task for PR."

The AI will:
1. Validate everything
2. Check tests
3. Verify documentation
4. Give you a go/no-go

---

## Step 7: Keep the Knowledge Base Honest

The knowledge base is trusted context — every flow loads it first. Facts drift as your project evolves (test framework swapped, a constraint relaxed, tasks finished outside the flows), and a stale KB quietly feeds every future flow wrong facts.

Two tools keep it honest:

- **Status** — ask your AI: "Show workflow status." Besides progress, it runs a read-only drift audit: task records that say `pending` for shipped features, malformed sign-off blocks, KB claims that contradict the codebase, and how long ago each KB file was last synced.
- **KB sync** — ask your AI: "Sync the knowledge base." It audits every KB claim and task record against the live codebase, shows you claim → reality → proposed fix, and applies corrections at review gates.

Run a KB sync whenever status flags drift, after work done outside the flows, or at the end of each milestone.

---

## Next Steps

- [Read how to customize](how-to-customize.md)
- [Read how to add custom flows](how-to-add-flow.md)
- [Read architecture explained](architecture-explained.md)

---

## Troubleshooting

### "Profile not detected"
The AI will ask you to choose a profile or create a custom one. Pick the closest match.

### "Flows not generated"
Check that `.ai-workflow/flows/` exists after the AI finishes initialization.

### "Design phase stuck"
Type `A` or `Accept` explicitly. The AI waits for your approval.

---

## Tips

- **Be specific** when describing tasks
- **Iterate** — it's okay to reject and refine
- **Update docs** as you go — they're living documents
- **Use diagrams** — they catch misunderstandings early
