# Playbook ↔ Skill Matrix

Which agent skills the most common playbooks rely on. Skills are persistent agent competencies; playbooks are stage-specific workflows.

| Playbook | Stage | Required skills | Optional skills |
|---|---|---|---|
| Discovery & Ideation | DISCOVER | requirement-extraction | codebase-navigation |
| Specification Authoring | DISCOVER | requirement-extraction | context-window-management |
| Architecture & Decomposition | ARCHITECT | codebase-navigation, dependency-assessment | context-window-management |
| Iterative Construction | CONSTRUCT | incremental-verification, error-interpretation | safe-refactoring |
| Test-First Engineering | CONSTRUCT | incremental-verification | error-interpretation |
| Context Orchestration | CONSTRUCT | context-window-management | multi-agent-collaboration |
| Contract-First APIs | CONSTRUCT | requirement-extraction | dependency-assessment |
| Threat Mitigation | HARDEN | production-awareness | dependency-assessment |
| Throughput Tuning | HARDEN | production-awareness | error-interpretation |
| Runtime Diagnostics | VALIDATE | error-interpretation | production-awareness |
| Fault Isolation & Recovery | VALIDATE | error-interpretation, incremental-verification | — |
| Peer Review Protocol | VALIDATE | codebase-navigation | human-escalation |
| Complexity Reduction | VALIDATE | safe-refactoring, incremental-verification | — |
| Version Control Discipline | RELEASE | incremental-verification | — |
| Pipeline Automation | RELEASE | production-awareness | — |
| Release Orchestration | RELEASE | production-awareness, human-escalation | — |
| Knowledge Capture | RELEASE | requirement-extraction | — |
| Observability & Ops | OPERATE | production-awareness, error-interpretation | human-escalation |
| Meta — How To Use | (cross) | — | all |

**Use this matrix to:** (1) load the right skills into an agent's profile before kicking off a stage, or (2) audit whether a missing skill is why a playbook keeps stalling.
