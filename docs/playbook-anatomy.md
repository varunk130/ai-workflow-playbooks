# Playbook Anatomy

This document describes the standard structure of a playbook. Follow this format when creating new playbooks.

## File Location

Every playbook lives at `playbooks/<playbook-name>/PLAYBOOK.md`.

The directory name uses lowercase-hyphen-separated words. The file is always named `PLAYBOOK.md` (uppercase).

## Standard Structure

```markdown
---
name: playbook-name-with-hyphens
trigger: When to activate this playbook. Use when [conditions].
---

# Playbook Title

## Objective
One or two sentences on what this playbook achieves and why it matters in production.

## Triggers
- Bullet list of conditions that activate this playbook
- Include "Do Not Use When" conditions for clarity

## Workflow
Numbered, ordered steps that the agent follows sequentially.
Each step should be specific and actionable.
"Run `npm test`" is a good step.
"Ensure quality" is not.

## Guardrails
Boundaries and constraints the agent must respect.
These are hard rules, not suggestions.

## Exit Criteria
- [ ] Checklist of observable evidence required before proceeding
- [ ] Each item should be verifiable (test output, file exists, metric threshold)

## Anti-Patterns
| Shortcut | Why It Fails |
|----------|-------------|
| Common excuse or shortcut | Concrete consequence of taking this shortcut |
```

## Design Principles

### Workflows, Not Reference Docs
Playbooks are step-by-step processes that agents execute in order. They are not encyclopedic reference material. If the agent needs reference material, link to a runbook.

### Specific Over General
Every instruction should be concrete enough that an agent can execute it without interpretation. Compare:
- **Vague**: "Write good tests"
- **Specific**: "Write a test that calls the function with boundary inputs and asserts the output matches expected values"

### Anti-Pattern Inoculation
Every playbook includes a table of common shortcuts and why they fail. This prevents agents from rationalizing their way around the workflow.

### Verifiable Exit Criteria
Each criterion should produce observable evidence:
- "All tests pass" → verifiable by running `npm test`
- "Code is clean" → not verifiable; replace with specific checks

### Progressive Loading
Core workflow stays in `PLAYBOOK.md`. Supporting material that exceeds ~100 lines goes in a separate file within the playbook directory or in `runbooks/` if it's cross-cutting.

## Supporting Files

If a playbook needs supplementary material:
- Place it in the same directory: `playbooks/<name>/supporting-topic.md`
- Reference it from the main PLAYBOOK.md
- Only create supporting files for content exceeding ~100 lines

Cross-cutting reference material belongs in `runbooks/`:
- `runbooks/testing-patterns.md`
- `runbooks/security-baseline.md`
- etc.
