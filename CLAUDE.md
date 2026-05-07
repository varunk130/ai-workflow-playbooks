# AI Workflow Playbooks — Project Context

This repository contains structured engineering playbooks designed for AI coding agents. Each playbook guides agents through a specific phase of the software delivery pipeline with executable workflows, exit criteria, and anti-pattern guards.

## Repository Layout

- `playbooks/` — 21 operational playbooks organized by pipeline stage
- `guardians/` — 4 specialist review personas (architecture, quality, security, operations)
- `runbooks/` — 5 cross-cutting operational checklists
- `lifecycle/` — Pipeline stage documentation and hooks
- `docs/` — Platform-specific setup guides
- `.claude/commands/` — 7 pipeline slash commands

## Pipeline Stages

DISCOVER → ARCHITECT → CONSTRUCT → HARDEN → VALIDATE → RELEASE → OPERATE

Each stage gates the next: an agent must satisfy a stage's exit criteria before proceeding. Skipping stages is an anti-pattern; a guardian review is required at HARDEN and VALIDATE.

## Conventions

- Playbook files are named `PLAYBOOK.md`
- Each playbook has: Objective, Triggers, Workflow, Guardrails, Exit Criteria, Anti-Patterns
- Guardians are specialist personas loaded for targeted review
- Runbooks are reference checklists loaded on demand
- All content is plain Markdown for maximum agent compatibility
