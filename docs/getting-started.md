# Getting Started with AI Workflow Playbooks

## Overview

AI Workflow Playbooks provides structured, executable workflows for AI coding agents. Each playbook guides an agent through a specific phase of software delivery with clear steps, exit criteria, and anti-pattern guards.

## Quick Start

### Option 1: Claude Code (Recommended)

```bash
git clone https://github.com/varunk130/ai-workflow-playbooks.git
claude --plugin-dir /path/to/ai-workflow-playbooks
```

Use pipeline commands directly:
```
/discover    — Explore problem space and write specifications
/architect   — Plan tasks and dependencies
/construct   — Build with tests
/harden      — Security and performance review
/validate    — Code review
/release     — Ship to production
/operate     — Set up monitoring
```

### Option 2: Copy Playbooks Manually

Playbooks are plain Markdown files. Copy the relevant `PLAYBOOK.md` files into your agent's context:

- **Cursor**: Copy into `.cursor/rules/`
- **GitHub Copilot**: Add to `.github/copilot-instructions.md`
- **Gemini CLI**: Load as context files
- **Windsurf**: Add to Windsurf rules configuration
- **Any other agent**: Paste into system prompt or rules

### Option 3: Reference Specific Playbooks

If you only need one workflow, load just that playbook. For example, to use test-first engineering, load only `playbooks/test-first-engineering/PLAYBOOK.md`.

## Recommended Starting Configuration

Start with these three playbooks for the highest impact:

1. **Specification Authoring** (`playbooks/specification-authoring/PLAYBOOK.md`)
   - Forces clear requirements before any code is written
   - Prevents the "build first, ask questions later" failure mode

2. **Test-First Engineering** (`playbooks/test-first-engineering/PLAYBOOK.md`)
   - Ensures every behavior is proven with a test
   - Catches regressions immediately

3. **Peer Review Protocol** (`playbooks/peer-review-protocol/PLAYBOOK.md`)
   - Structured multi-axis code review
   - Catches issues across correctness, security, and performance

## Adding Specialist Reviews

Load guardians when you need focused expertise:

```
Load guardians/architecture-guardian.md for system design review
Load guardians/quality-guardian.md for test coverage assessment
Load guardians/security-guardian.md for security audit
Load guardians/operations-guardian.md for deployment readiness
```

## Loading Reference Material

Runbooks provide detailed checklists loaded on demand:

```
Load runbooks/testing-patterns.md for test examples and patterns
Load runbooks/security-baseline.md for OWASP and auth checklists
Load runbooks/performance-baseline.md for Core Web Vitals and optimization
Load runbooks/accessibility-baseline.md for WCAG 2.1 AA compliance
Load runbooks/incident-response.md for production incident procedures
```

## Next Steps

- Read the [Playbook Anatomy](playbook-anatomy.md) guide to understand how playbooks are structured
- Check platform-specific guides for your AI agent
- Start using `/discover` on your next feature
