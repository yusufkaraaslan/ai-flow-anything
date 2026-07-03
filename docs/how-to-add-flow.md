# How to Add a Flow

Extend ai-flow-anything with custom flow types.

---

## When to Add a Flow

Add a flow when:
- Your workflow has steps not covered by existing flows
- Your team has unique processes
- You want to automate a recurring task

Examples:
- Security audit flow
- Performance optimization flow
- Database migration flow
- Localization flow
- Accessibility audit flow

---

## Flow Structure

```markdown
# Flow: {Name}

## Purpose
What this flow does.

## When to Use
Conditions for using this flow.

## Input
What the flow needs.

## Output
What the flow produces.

## Prerequisites
What must be true before running.

## Phases

### Phase 1: {Name}
**Purpose:** What this phase does.

**Steps:**
1. Step description
2. Step description

**Artifacts:**
- File path and description

**Gate:** [A]ccept / [F]eedback / [R]eject

### Phase 2: {Name}
...

## Customization
How users can customize.
```

---

## Example: Security Audit Flow

```markdown
# Flow: Security Audit

## Purpose
Audit code for security vulnerabilities.

## When to Use
Before major releases or after security incidents.

## Input
- Source code
- Dependency list
- API endpoints

## Output
- Security report
- Fix recommendations
- Updated task flow files

## Prerequisites
- Code is committed
- Dependencies are listed

## Phases

(Following the canonical 2-phase, 2-gate pattern.)

### Phase 1: AUDIT *(STANDARD gate)*
**Purpose:** Scan, categorize, and remediate findings as one reviewable package.

**Sub-tasks (auto-proceed):**

**1.1 SCAN** — Find potential vulnerabilities:
- Search for hardcoded secrets (API keys, passwords)
- Check for SQL injection risks
- Verify input validation
- Check authentication coverage
- Review authorization logic

**1.2 ASSESS** — Categorize each finding by severity:
- Critical: Data exposure, RCE
- High: Auth bypass, injection
- Medium: XSS, CSRF
- Low: Info disclosure
Map to OWASP Top 10 and compliance requirements (GDPR, SOC2).

**1.3 REMEDIATE** — Generate + apply fixes:
- Generate fix for each finding (present with before/after)
- Apply approved fixes
- Re-scan to verify

**Artifacts (presented at gate):**
- `flow-storage/project/security/audit-{date}.md`
- Risk assessment matrix
- Fixed code (diffs)
- `flow-storage/project/security/remediation-{date}.md`

**Gate:** Audit + remediation review — [A]ccept / [F]eedback / [R]eject

### Phase 2: COMMIT *(CRITICAL gate)*
**Purpose:** Document the final security posture and commit.

**Sub-tasks (auto-proceed):**

**2.1 DOCUMENT** — Update security documentation, record decisions (won't fix, accepted risk), update threat model, schedule next audit.

**2.2 COMMIT** *(executes only after the gate is accepted)* — Stage updated security docs + fixed code; commit `security: audit + remediation {date}`.

**Artifacts (presented at gate):**
- Updated security docs
- `flow-storage/project/security/audit-log.md`
- Staged file list
- Proposed commit message

**Gate:** Commit review — [A]ccept / [F]eedback / [R]eject
```

---

## Adding to Your Project

1. Create file `.ai-workflow/flows/{name}-flow.md`
2. Write flow following structure above
3. Invoke from your AI assistant using whatever style your platform supports — slash command on Claude Code (`/{name}-flow`) and OpenCode, natural language anywhere else ("run the {name} flow for X"). Add the new flow to the intent table in `.ai-workflow/instructions.md` so the AI recognises its trigger phrases.

---

## Adding to Profile

To share with profile users:

1. Create file `profiles/{type}/skeletons/{name}-flow.md`
2. Add the flow to the profile's README capability table
3. Document the new flow's intent in `instructions.md` if it should be discoverable globally; otherwise the profile's skeletons surface it during initialization

To add a flow to the **standard set** (every profile), additionally:

4. Add its row to the flow-types table and gate-cadence table in `universal/workflow-structure.md`, plus a phase-definition section
5. Add a canonical skeleton to `profiles/generic/skeletons/` and a slim defer-pattern overlay to every other profile
6. Sweep the duplicated flow lists and file counts: `README.md`, `docs/`, `platforms/{tool}/` wrappers and READMEs, `instructions.md` Steps 7–8
7. Bump `VERSION`

---

## Tips

- **Start simple** — 2-3 phases are enough
- **Reuse patterns** — Copy structure from existing flows
- **Be specific** — Tell AI exactly what to check
- **Include gates** — Every phase needs review
- **Document output** — What files are created
- **Test it** — Run on real code before relying on it

---

## Common Custom Flows

| Flow | Purpose | When |
|--------|---------|------|
| **security-audit** | Find vulnerabilities | Before releases |
| **perf-optimize** | Performance improvements | When slow |
| **db-migrate** | Database migrations | Schema changes |
| **i18n** | Internationalization | Adding languages |
| **a11y-audit** | Accessibility check | Compliance |
| **refactor** | Code refactoring | Tech debt |
| **onboard** | New developer setup | Team growth |
| **release** | Release checklist | Deployments |

---

## Integration with Existing Flows

Custom flows can integrate with the main workflow:

```markdown
## Integration

Run this flow after the implement flow and before the PR flow:

```
design flow  →  implement flow  →  security-audit flow  →  PR flow
```

If security audit fails, fix issues before PR.
```
