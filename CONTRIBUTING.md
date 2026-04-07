# Contributing to AI Workflow Playbooks

Thank you for helping improve these playbooks. This guide covers quality standards, structure conventions, and the review process.

## Playbook Quality Standards

Every playbook must be:

- **Executable** — Step-by-step workflows the agent follows, not reference articles
- **Verifiable** — Clear exit criteria with observable evidence (test output, build logs, metrics)
- **Grounded** — Based on real engineering practices, not theoretical ideals
- **Focused** — Only the content necessary to guide the agent through the workflow

## Playbook Structure

Every playbook lives in `playbooks/<playbook-name>/PLAYBOOK.md` and follows this skeleton:

```markdown
---
name: playbook-name-with-hyphens
trigger: When to activate this playbook. Use when [conditions].
---

# Playbook Title

## Objective
One or two sentences on what this playbook achieves and why it matters.

## Triggers
- When this playbook should activate
- When it should NOT activate

## Workflow
Numbered steps with specific, actionable instructions.

## Guardrails
Boundaries and constraints the agent must respect.

## Exit Criteria
- [ ] Checklist of required evidence before proceeding

## Anti-Patterns
| Shortcut | Why It Fails |
|----------|-------------|
| Common excuse | Concrete consequence |
```

## What Belongs Where

| Content Type | Location |
|-------------|----------|
| Core workflow | `playbooks/<name>/PLAYBOOK.md` |
| Supporting material (>100 lines) | `playbooks/<name>/<topic>.md` |
| Cross-cutting checklists | `runbooks/<name>.md` |
| Specialist personas | `guardians/<name>.md` |
| Pipeline commands | `.claude/commands/<name>.md` |
| Setup guides | `docs/<name>.md` |

## Naming Conventions

- Directories: `lowercase-hyphen-separated`
- Playbook files: Always `PLAYBOOK.md` (uppercase)
- Supporting files: `lowercase-hyphen-separated.md`
- Guardian files: `<role>-guardian.md`

## Anti-Patterns in Contributions

- **Don't duplicate** — Reference other playbooks instead of repeating their content
- **Don't generalize** — "Write good tests" is not a step. "Run `npm test` and verify zero failures" is
- **Don't bloat** — If a supporting file would be under 50 lines, inline it
- **Don't skip anti-patterns** — Every playbook needs its "common shortcuts" table

## Pull Request Process

1. Create a branch from `main`
2. Add or modify playbook content
3. Ensure the playbook follows the structure above
4. Test with at least one AI agent to verify it produces useful guidance
5. Open a PR with a description of what changed and why
