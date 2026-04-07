---
name: meta-how-to-use
trigger: Use when onboarding to the AI Workflow Playbooks system or when unsure which playbook to activate.
---

# How to Use AI Workflow Playbooks

## Objective
Guide agents and users through the playbook system so they can select and apply the right playbook at the right time. This is the entry point for new users.

## The Pipeline Model

Every software task moves through a pipeline of stages. Not every task requires every stage, but skipping stages should be a conscious decision, not an oversight.

```
DISCOVER → ARCHITECT → CONSTRUCT → HARDEN → VALIDATE → RELEASE → OPERATE
```

### Quick Selection Guide

**"I have a vague idea"** → Start at DISCOVER
- Use Discovery & Ideation to explore the problem
- Then Specification Authoring to document requirements

**"I have a spec, ready to build"** → Start at ARCHITECT
- Use Architecture & Decomposition to plan tasks
- Then proceed through CONSTRUCT playbooks

**"I'm writing code right now"** → You're in CONSTRUCT
- Use Iterative Construction for the build loop
- Use Test-First Engineering for test strategy
- Use Interface Craftsmanship for frontend work
- Use Contract-First APIs for backend interfaces

**"Code is written, need to verify"** → You're in VALIDATE
- Use Peer Review Protocol for code review
- Use Runtime Diagnostics for browser verification
- Use Fault Isolation & Recovery for debugging

**"Need to harden before shipping"** → You're in HARDEN
- Use Threat Mitigation for security review
- Use Throughput Tuning for performance optimization

**"Ready to ship"** → You're in RELEASE
- Use Version Control Discipline for clean commits
- Use Pipeline Automation for CI/CD
- Use Release Orchestration for the go-live process

**"It's live, now what?"** → You're in OPERATE
- Use Observability & Ops for monitoring and incident readiness

## Loading Playbooks

### Minimal Set (Start Here)
For most tasks, these three playbooks cover the critical gaps:
1. **Specification Authoring** — Prevents building the wrong thing
2. **Test-First Engineering** — Prevents shipping broken things
3. **Peer Review Protocol** — Catches what you missed

### Full Pipeline
For production features, load the complete pipeline stage by stage as you progress.

### On-Demand Loading
Guardians and runbooks are loaded only when needed:
- Load a **Guardian** when you need a specialist review (security, architecture, quality, ops)
- Load a **Runbook** when you need a detailed checklist (testing patterns, security baseline, etc.)

## Using Pipeline Commands

If using Claude Code with the plugin installed:

| Command | Stage | What It Does |
|---------|-------|-------------|
| `/discover` | DISCOVER | Runs Discovery & Ideation workflow |
| `/architect` | ARCHITECT | Runs Architecture & Decomposition workflow |
| `/construct` | CONSTRUCT | Runs Iterative Construction + Test-First Engineering |
| `/harden` | HARDEN | Runs Threat Mitigation + Throughput Tuning |
| `/validate` | VALIDATE | Runs Peer Review Protocol |
| `/release` | RELEASE | Runs Release Orchestration workflow |
| `/operate` | OPERATE | Runs Observability & Ops workflow |

## Principles

1. **Follow the workflow steps** — Playbooks are processes, not reference docs. Execute them in order.
2. **Meet exit criteria** — Don't proceed to the next stage until exit criteria are satisfied with evidence.
3. **Document deviations** — If you skip a step, note why. "Not applicable" is valid; silence is not.
4. **Load progressively** — Start with what you need; add playbooks as the task demands.
5. **Guardians are reviewers** — They provide a specialist perspective on your work, not an additional workflow.

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| Loading all 21 playbooks at once | Context overload; the agent loses focus |
| Skipping DISCOVER and ARCHITECT for "simple" features | Simple features have a way of growing; specs prevent scope creep |
| Treating playbooks as suggestions | The value is in the discipline; optional steps get skipped |
| Using only CONSTRUCT, never VALIDATE | Shipping unreviewed code is a reliability risk |
| Skipping OPERATE entirely | Production issues hit blind without monitoring |
