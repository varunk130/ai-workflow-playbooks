---
name: release-orchestration
trigger: When preparing a production release, managing a staged rollout, or evaluating go/no-go criteria
---

# Release Orchestration

Staged rollouts, feature flags, and go-live checklists for confident releases.

## Objective

Ship changes to production with controlled risk, fast rollback capability, and
clear accountability. Every release follows a repeatable process that catches
problems before they reach all users, and every participant knows the criteria
for proceeding or aborting.

## Triggers

- A feature or set of changes is ready to move from staging to production.
- A high-risk change (database migration, infrastructure, third-party integration)
  is approaching deployment.
- A rollback is needed due to degraded metrics after a release.
- A scheduled release window is approaching.

## Workflow

### 1. Complete the Pre-Launch Checklist

Before any production deployment, verify every item:

- [ ] All CI/CD pipeline stages are green (lint, test, build, security scan).
- [ ] Security scan shows no unmitigated critical or high findings.
- [ ] Documentation is updated (API docs, README, changelog, runbook).
- [ ] Monitoring and alerting are configured for new or changed behavior.
- [ ] Feature flags are in place for any change that should be gated.
- [ ] Rollback procedure is documented and has been dry-run in staging.
- [ ] Stakeholders (product, support, on-call) are notified of the release window.
- [ ] Database migrations are backward-compatible and have been tested in staging.

### 2. Manage Feature Flag Lifecycle

Feature flags control exposure and provide instant rollback without redeployment.

```
Create flag --> Enable for team --> Percentage rollout --> Full enable --> Remove flag
```

| Phase | Audience | Purpose |
|---|---|---|
| **Create** | Nobody (flag off) | Code merged safely behind a gate |
| **Team internal** | Engineering + QA | Validate in production environment |
| **Percentage rollout** | 5-10% of users | Detect issues at small scale |
| **Full enable** | 100% of users | General availability |
| **Remove flag** | N/A | Clean up within one sprint of full rollout |

### 3. Execute the Staged Rollout

Deploy the artifact and increase exposure incrementally:

1. **Canary** -- Route a single instance or ~1% of traffic to the new version.
   Monitor error rates, latency, and resource usage for 10-15 minutes.
2. **10% rollout** -- Expand to 10% of users. Watch business metrics (conversion,
   checkout success) alongside system metrics.
3. **50% rollout** -- Expand to half. Run for a meaningful traffic window (at least
   one peak period) to surface load-dependent issues.
4. **100% rollout** -- Full deployment. Continue monitoring for 24 hours before
   declaring the release stable.

At **any stage**, if metrics breach the rollback criteria, halt and revert.

### 4. Define Go / No-Go Criteria

The release lead evaluates these signals at each rollout stage:

| Signal | Go | No-Go |
|---|---|---|
| Error rate | Within baseline +0.1% | Exceeds baseline +0.5% |
| P99 latency | Within baseline +10% | Exceeds baseline +25% |
| Health checks | All passing | Any critical check failing |
| Business KPIs | Stable or improving | Statistically significant regression |
| On-call alerts | No new pages | New page triggered by the release |

Decision authority: the **release lead** makes the call. Ambiguous cases default
to **no-go** until the signal is clarified.

### 5. Rollback Procedures

- Rollback is a **first-class, automated action** -- never a heroic manual effort.
- Disable the feature flag immediately to halt user exposure.
- Redeploy the previous artifact version if the flag alone is insufficient.
- For database changes, apply the reverse migration only if backward-compatible;
  otherwise, fix forward with an expedited patch.
- Notify stakeholders of the rollback and estimated time to resolution.

### 6. Post-Launch Verification

After reaching 100% and holding for 24 hours:

- [ ] All monitoring dashboards show nominal behavior.
- [ ] No elevated error rates, latency spikes, or resource anomalies.
- [ ] Customer support has not flagged new issue trends.
- [ ] Feature flag is scheduled for removal within the current sprint.
- [ ] Release notes are published (changelog, internal announcement).
- [ ] A brief retrospective is held for high-risk releases.

## Guardrails

- **No production deploys without a passing pre-launch checklist.**
- **Feature flags are mandatory** for any user-facing change that cannot be trivially
  reverted by redeployment.
- **Rollback must be executable in under 5 minutes.**
- **No skipping rollout stages** -- canary is never optional.
- **After-hours deploys require on-call coverage** and explicit approval from the
  release lead.

## Exit Criteria

- [ ] Pre-launch checklist is complete with all items verified.
- [ ] Staged rollout reached 100% without breaching no-go thresholds.
- [ ] Post-launch verification confirms nominal behavior for 24 hours.
- [ ] Feature flags used in this release are scheduled for removal.
- [ ] Release notes and changelog are published.
- [ ] Retrospective captured for high-risk releases.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Alternative |
|---|---|---|
| YOLO deploy straight to 100% | No early warning; full blast radius on failure | Canary then staged percentage rollout |
| Feature flags left on indefinitely | Dead code paths, logic complexity, stale risk | Remove within one sprint of full enablement |
| Rollback requires manual SSH and scripts | Slow, error-prone, stressful under pressure | Automated rollback via pipeline or flag toggle |
| No pre-launch checklist | Forgotten migrations, missing alerts, stale docs | Enforce checklist as a merge/deploy gate |
| Deploying on Friday afternoon | Limited monitoring window, skeleton on-call crew | Deploy early in the week with full team available |
| Skipping post-launch monitoring | Silent regressions compound over days | Monitor for 24 hours; verify dashboards and KPIs |
