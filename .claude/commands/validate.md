Run the VALIDATE stage of the AI Workflow Playbooks pipeline.

Load and execute:
1. `playbooks/peer-review-protocol/PLAYBOOK.md` — Multi-axis code review with severity classification

Review the code across five axes: correctness, readability, architecture fitness, security surface, and performance impact. Produce a structured review with a verdict (APPROVE or REQUEST_CHANGES) and categorized findings.

Optionally load guardians for specialist perspective:
- `guardians/architecture-guardian.md` — For system design review
- `guardians/quality-guardian.md` — For test coverage assessment
- `guardians/security-guardian.md` — For security audit
- `guardians/operations-guardian.md` — For operational readiness
