# AI Workflow Playbooks — Pipeline Reference

## The 7-Stage Delivery Pipeline

```
Stage 1    Stage 2      Stage 3       Stage 4    Stage 5     Stage 6    Stage 7
DISCOVER → ARCHITECT → CONSTRUCT  → HARDEN  → VALIDATE → RELEASE → OPERATE
```

### Stage 1: DISCOVER
**Goal**: Understand the problem and define what to build.

| Playbook | Command |
|----------|---------|
| Discovery & Ideation | `/discover` |
| Specification Authoring | `/discover` |

**Key Output**: Discovery brief + complete specification

### Stage 2: ARCHITECT
**Goal**: Design the system and plan the work.

| Playbook | Command |
|----------|---------|
| Architecture & Decomposition | `/architect` |

**Key Output**: Ordered task list with dependencies and acceptance criteria

### Stage 3: CONSTRUCT
**Goal**: Build incrementally with quality baked in.

| Playbook | Command |
|----------|---------|
| Iterative Construction | `/construct` |
| Test-First Engineering | `/construct` |
| Context Orchestration | (loaded as needed) |
| Interface Craftsmanship | (loaded for frontend work) |
| Contract-First APIs | (loaded for API work) |

**Key Output**: Working, tested code in atomic commits

### Stage 4: HARDEN
**Goal**: Armor the system against threats and inefficiency.

| Playbook | Command |
|----------|---------|
| Threat Mitigation | `/harden` |
| Throughput Tuning | `/harden` |

**Key Output**: Security audit + performance assessment

### Stage 5: VALIDATE
**Goal**: Prove the system meets standards through review.

| Playbook | Command |
|----------|---------|
| Runtime Diagnostics | (loaded for browser testing) |
| Fault Isolation & Recovery | (loaded for debugging) |
| Peer Review Protocol | `/validate` |
| Complexity Reduction | (loaded for simplification) |

**Key Output**: Review verdict with categorized findings

### Stage 6: RELEASE
**Goal**: Ship with confidence and traceability.

| Playbook | Command |
|----------|---------|
| Version Control Discipline | (always active) |
| Pipeline Automation | (CI/CD setup) |
| Sunset & Evolution | (deprecation work) |
| Knowledge Capture | (documentation) |
| Release Orchestration | `/release` |

**Key Output**: Staged production deployment with rollback plan

### Stage 7: OPERATE
**Goal**: Monitor, learn, and improve after deployment.

| Playbook | Command |
|----------|---------|
| Observability & Ops | `/operate` |

**Key Output**: Monitoring, alerting, dashboards, and incident readiness

## Stage Transitions

Each stage produces artifacts that serve as inputs to the next stage:

```
DISCOVER produces → Specification
ARCHITECT consumes Specification → produces Task Sequence
CONSTRUCT consumes Task Sequence → produces Working Code
HARDEN consumes Working Code → produces Hardened Code
VALIDATE consumes Hardened Code → produces Review Verdict
RELEASE consumes Review Verdict → produces Production Deployment
OPERATE consumes Production Deployment → produces Operational Readiness
```

## Not Every Task Needs Every Stage

- **Bug fix**: CONSTRUCT → VALIDATE → RELEASE
- **New feature**: All 7 stages
- **Performance issue**: HARDEN → VALIDATE → RELEASE
- **Security incident**: HARDEN → RELEASE → OPERATE
- **Documentation update**: RELEASE (just commit and ship)
