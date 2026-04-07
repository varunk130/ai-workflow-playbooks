---
name: observability-and-ops
trigger: Use after deployment to establish monitoring, alerting, and incident response readiness. Activate during the OPERATE stage.
---

# Observability & Ops

## Objective
Ensure every deployed system is observable, alertable, and operable from day one. Production code without monitoring is a silent failure waiting to happen. This playbook extends the pipeline beyond deployment into the ongoing operational lifecycle.

## Triggers
- A new service or feature has been deployed to production
- An existing service lacks monitoring or alerting
- After an incident exposes an observability gap
- Setting up a new production environment

## Do Not Use When
- The system is still in local development with no deployment
- You're working on a prototype that won't reach production

## Workflow

### Step 1: Define the Observability Contract
For every deployed service, document:

```
SERVICE: [name]
HEALTH ENDPOINT: [URL for liveness/readiness checks]
KEY METRICS:
  - Request rate (requests/second)
  - Error rate (% of 5xx responses)
  - Latency (p50, p95, p99 response times)
  - Saturation (CPU, memory, disk, connection pool usage)
LOG FORMAT: [structured JSON with correlation IDs]
TRACE CONTEXT: [distributed tracing headers propagated]
```

This follows the RED method (Rate, Errors, Duration) for request-driven services and USE method (Utilization, Saturation, Errors) for infrastructure resources.

### Step 2: Instrument the Application
Add observability at three layers:

**Metrics** — Counters, gauges, histograms:
- Request count by endpoint, method, and status code
- Response latency histograms (p50, p95, p99)
- Active connection counts
- Queue depths and processing times
- Business metrics (signups, orders, completions)

**Logs** — Structured, contextual, leveled:
- Use structured logging (JSON) with consistent field names
- Include correlation/request IDs in every log line
- Log at appropriate levels: ERROR for failures, WARN for degradation, INFO for significant events, DEBUG for troubleshooting
- Never log sensitive data (passwords, tokens, PII)

**Traces** — Distributed request flow:
- Propagate trace context across service boundaries
- Instrument external calls (database queries, HTTP requests, message queues)
- Record span metadata (query text, endpoint, status)

### Step 3: Configure Alerting
Alerts should be actionable — every alert must have a response procedure.

| Alert Type | Threshold | Severity | Response |
|-----------|-----------|----------|----------|
| Error rate spike | >5% of requests are 5xx for 5 minutes | Critical | Page on-call; check recent deployments |
| Latency degradation | p99 > 2x baseline for 10 minutes | Warning | Investigate; check downstream dependencies |
| Resource saturation | CPU/memory > 85% for 15 minutes | Warning | Scale or investigate leak |
| Health check failure | 3 consecutive failures | Critical | Restart service; check dependencies |
| Certificate expiry | <14 days until expiration | Warning | Renew certificate |

Rules for good alerts:
- Every alert has a documented runbook link
- Alerts fire on symptoms (user impact), not causes (CPU usage alone)
- Tune thresholds to avoid alert fatigue — if it fires daily and gets ignored, it's broken
- Group related alerts to reduce noise

### Step 4: Build Dashboards
Create operational dashboards with progressive detail:

**Level 1 — Service Health** (glanceable):
- Overall request rate, error rate, latency
- Service up/down status
- Active incidents

**Level 2 — Component Detail** (investigative):
- Per-endpoint metrics
- Database query performance
- External dependency latency and error rates
- Cache hit ratios

**Level 3 — Debug** (root cause analysis):
- Trace waterfall views
- Log search by correlation ID
- Resource utilization over time

### Step 5: Establish Incident Response Readiness
Before considering a service production-ready:

1. **Runbook exists** — Documented procedures for common failure scenarios
2. **On-call rotation is defined** — Someone is responsible for responding
3. **Rollback procedure is tested** — You can revert to the previous version quickly
4. **Communication channels are set** — Where incidents are reported and coordinated
5. **Postmortem template is ready** — How learnings are captured after resolution

### Step 6: Validate Observability
Verify the system works by intentionally testing:
- Trigger a known error → verify it appears in logs and metrics
- Simulate high latency → verify alerting fires
- Trace a request end-to-end → verify spans connect across services
- Search logs by correlation ID → verify the full request lifecycle is visible

## Guardrails
- Never deploy a service without a health check endpoint
- Logs must not contain PII, credentials, or authentication tokens
- Alerts without runbooks are incomplete — fix the alert or write the runbook
- Dashboard access must be available to anyone who might respond to an incident
- Monitoring infrastructure must itself be monitored (meta-monitoring)

## Exit Criteria
- [ ] Health check endpoint exists and is monitored
- [ ] Metrics cover request rate, error rate, and latency
- [ ] Structured logging is in place with correlation IDs
- [ ] Distributed tracing propagates across service boundaries
- [ ] Alerting rules are configured with documented thresholds
- [ ] Every alert links to a runbook
- [ ] Operational dashboards are built at health and detail levels
- [ ] Incident response process is documented
- [ ] Rollback procedure is tested and verified
- [ ] Observability has been validated with intentional test failures

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| "We'll add monitoring after launch" | You can't debug what you can't see; first production issues hit blind |
| Alerting on every metric | Alert fatigue — the team ignores alerts, including real ones |
| Unstructured log output | Impossible to search, filter, or correlate across requests |
| Dashboards only the author understands | Useless during incidents when someone else is responding |
| No runbooks for alerts | On-call responders waste time figuring out what to do |
| Logging sensitive data for debugging | Compliance violation and security risk |
