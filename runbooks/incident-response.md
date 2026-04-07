# Incident Response Runbook

Operational procedures for handling production incidents. Load this when an incident occurs or when setting up incident readiness.

## Severity Levels

| Level | Definition | Response Time | Examples |
|-------|-----------|--------------|---------|
| SEV-1 | Service completely down; all users affected | Immediate (page on-call) | Database failure, full outage, data loss |
| SEV-2 | Major feature broken; many users affected | Within 15 minutes | Auth system down, payment failures, significant data errors |
| SEV-3 | Minor feature degraded; some users affected | Within 1 hour | Slow queries, intermittent errors, UI rendering issues |
| SEV-4 | Cosmetic or minor issue; workaround available | Next business day | Typo, minor styling bug, non-critical log noise |

## Incident Lifecycle

### Phase 1: Detection & Triage (First 5 Minutes)

1. **Confirm the incident** — Verify the alert or report is real, not a false positive
2. **Assess severity** — Use the severity table above
3. **Declare the incident** — Post in the incident channel with:
   ```
   INCIDENT: [Brief description]
   SEVERITY: [SEV-1/2/3/4]
   IMPACT: [Who is affected and how]
   COMMANDER: [Name of incident commander]
   STATUS: Investigating
   ```
4. **Assign roles**:
   - **Incident Commander** — Coordinates response, owns communication
   - **Technical Lead** — Investigates root cause and implements fix
   - **Communications Lead** — Updates stakeholders and status page

### Phase 2: Investigation (Minutes 5-30)

1. **Check recent changes** — Was anything deployed in the last hour?
2. **Review dashboards** — Error rates, latency, resource utilization
3. **Check dependencies** — Are downstream services healthy?
4. **Search logs** — Filter by error level and time window around incident start
5. **Correlate** — Match the incident timeline with deployment, config changes, or traffic spikes

### Phase 3: Mitigation (As Soon As Root Cause is Identified)

Choose the fastest path to restore service:

| Strategy | When to Use |
|----------|-------------|
| **Rollback** | Recent deployment caused the issue; previous version was stable |
| **Feature flag disable** | New feature is the culprit; can be toggled off without full rollback |
| **Scale up** | Traffic or load exceeds capacity; buying time while investigating |
| **Redirect traffic** | One region or instance is affected; route around it |
| **Hotfix** | Root cause is clear and fix is small and safe |

### Phase 4: Resolution

1. **Verify mitigation** — Confirm metrics return to normal baselines
2. **Monitor for recurrence** — Watch for 30 minutes after mitigation
3. **Update status** — Post resolution to incident channel and status page:
   ```
   INCIDENT RESOLVED: [Brief description]
   ROOT CAUSE: [One-sentence summary]
   MITIGATION: [What was done to restore service]
   DURATION: [Total incident duration]
   FOLLOW-UP: Postmortem scheduled for [date]
   ```
4. **Stand down** — Release the incident team

### Phase 5: Postmortem (Within 48 Hours)

Write a blameless postmortem covering:

```
## Incident Postmortem: [Title]
Date: [Date]
Duration: [Start time — End time]
Severity: [SEV-X]
Commander: [Name]

## Summary
[2-3 sentences describing what happened]

## Timeline
[Chronological list of events with timestamps]

## Root Cause
[Technical explanation of why the incident occurred]

## Impact
- Users affected: [count or percentage]
- Revenue impact: [if applicable]
- Data impact: [if applicable]

## What Went Well
- [Things that helped detect or resolve the incident quickly]

## What Went Wrong
- [Things that delayed detection or resolution]

## Action Items
- [ ] [Action] — Owner: [name] — Due: [date]
- [ ] [Action] — Owner: [name] — Due: [date]

## Lessons Learned
[Key takeaways for the team]
```

## Communication Templates

### Status Page Update
```
[Investigating] We are aware of issues with [service] and are investigating.
[Identified] The issue has been identified as [brief cause]. We are working on a fix.
[Monitoring] A fix has been applied. We are monitoring for stability.
[Resolved] The issue has been resolved. [Brief description of resolution].
```

### Stakeholder Update (for SEV-1/SEV-2)
```
Subject: [SERVICE] Incident Update — [Status]

Current Status: [Investigating / Mitigating / Resolved]
Impact: [Who is affected and how]
ETA: [Estimated time to resolution, or "investigating"]
Next Update: [When the next update will be sent]
```

## Readiness Checklist

- [ ] Incident communication channel exists and is monitored
- [ ] On-call rotation is defined with clear escalation paths
- [ ] Status page is set up and updatable within minutes
- [ ] Rollback procedure is documented and tested
- [ ] Dashboards are accessible to the incident response team
- [ ] Postmortem template is readily available
- [ ] Action items from previous postmortems are tracked to completion
