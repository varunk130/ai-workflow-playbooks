---
name: pipeline-automation
trigger: When setting up CI/CD, adding quality gates, or diagnosing slow pipelines
---

# Pipeline Automation

Shift-left CI/CD with quality gates that keep the fast lane open and the bar high.

## Objective

Automate the path from committed code to production so that every change is validated
quickly, consistently, and without manual ceremony. Catch defects as early and cheaply
as possible. Pipelines should be a developer's safety net, not a bottleneck.

## Triggers

- A new repository or service is being bootstrapped.
- An existing pipeline exceeds the 10-minute feedback target.
- A production incident traces back to a gap in automated validation.
- A new quality dimension (security, accessibility, performance) must be enforced.

## Workflow

### 1. Design the Stage Sequence

Run stages in order; fail fast so developers get signal within minutes.

```
lint --> unit-test --> build --> integration-test --> security-scan --> deploy
```

- **Lint** -- static analysis, formatting, type-checking.
- **Unit Test** -- fast, isolated tests; aim for < 2 minutes.
- **Build** -- compile artifacts, container images, or bundles.
- **Integration Test** -- service-level contracts, database migrations.
- **Security Scan** -- dependency audit, SAST, secret detection.
- **Deploy** -- push to the target environment (dev, staging, or production).

### 2. Define Quality Gates

Each stage boundary is a gate. A gate specifies the conditions that **must** pass
before the pipeline advances.

| Gate | Required Signal |
|---|---|
| Post-lint | Zero lint errors, formatting clean |
| Post-unit-test | All tests pass, coverage >= threshold |
| Post-build | Artifact produced, size within budget |
| Post-security-scan | No critical/high CVEs, no leaked secrets |
| Pre-production deploy | Staging smoke tests pass, change approved |

### 3. Keep Feedback Fast

- Target total pipeline time: **< 10 minutes** for the critical path.
- Parallelize independent stages (e.g., lint and unit test simultaneously).
- Cache dependencies, layers, and build artifacts aggressively.
- Run expensive tests (E2E, performance) on a separate non-blocking track
  and report results asynchronously.

### 4. Use Feature Flags to Decouple Deploy from Release

- Deploy dark code behind flags so the artifact reaches production without
  user-facing impact.
- Separate the **deployment** (technical act) from the **release** (business decision).
- Remove flags within one sprint of full rollout to prevent flag debt.

### 5. Automate Rollback

- Every deployment records the previous artifact version.
- Rollback is a single command or automatic on health-check failure.
- Canary or blue-green strategies limit blast radius.
- Runbooks for manual rollback exist as a fallback.

### 6. Promote Through Environments

```
dev --> staging --> production
```

- **Dev** -- deployed on every push to trunk; used for integration validation.
- **Staging** -- mirrors production config; used for final acceptance and load tests.
- **Production** -- deployed after staging gate clears; monitored for anomalies.
- Use the same artifact across all environments; only configuration changes.

## Guardrails

- **Pipeline definitions live in version control** alongside application code.
- **No manual steps** between commit and production except explicit approval gates.
- **Secrets are injected at runtime** via the CI platform's secret store, never
  hard-coded in pipeline files.
- **Pipeline changes require review** through the same PR process as application code.
- **Flaky tests are quarantined immediately**, not ignored or retried silently.

## Exit Criteria

- [ ] Pipeline runs end-to-end on every push to trunk.
- [ ] Total feedback time for the critical path is under 10 minutes.
- [ ] Quality gates block promotion when thresholds are breached.
- [ ] Rollback can be executed in under 5 minutes.
- [ ] Feature flags decouple deployment from release for risky changes.
- [ ] The same artifact is promoted across dev, staging, and production.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Alternative |
|---|---|---|
| Manual deploy rituals | Error-prone, slow, undocumented | Fully automated pipeline with approval gate |
| Ignoring flaky tests | Erodes trust, masks real failures | Quarantine flaky tests; fix or delete promptly |
| Building different artifacts per env | "Works on staging" drift | Build once, configure per environment |
| Pipeline > 20 minutes | Developers stop waiting, batch changes | Parallelize, cache, split slow tests out |
| Security scans only in production | Late discovery, expensive remediation | Shift left: scan in CI on every commit |
| Flag debt (stale feature flags) | Dead code paths, confusing logic | Remove flags within one sprint of full rollout |
