---
name: production-awareness
capability: Think about production consequences at every stage of development, not just at deployment time.
---

# Production Awareness

## What This Skill Enables
An agent that considers how code will behave in production — under real load, with real data, facing real failures — while writing it, not after shipping it. Without this skill, agents write code that works in development but fails under production conditions: high concurrency, network failures, malformed data, resource exhaustion, and hostile input.

## Core Competencies

### 1. Production Mental Model
While writing any code, ask these questions:

| Question | Why It Matters |
|----------|---------------|
| What happens at 100x current load? | Linear algorithms become bottlenecks; database connections exhaust |
| What happens when this external call fails? | Network timeouts, retries, and circuit breakers become essential |
| What happens with unexpected input? | User-generated content includes Unicode, injection attempts, and extreme lengths |
| What happens when disk/memory fills up? | Unbounded logs, caches, and queues cause cascading failures |
| What happens during a deploy? | In-flight requests, database migrations, and cache invalidation need handling |

### 2. Failure Mode Awareness
Every external dependency will eventually fail. Design for it:

**Network calls**: Always have a timeout, retry policy, and fallback behavior
```
GOOD: fetch(url, { signal: AbortSignal.timeout(5000) })
       .catch(err => fallbackResponse)

BAD:  fetch(url)  // hangs forever if the server doesn't respond
```

**Database queries**: Handle connection pool exhaustion, slow queries, and deadlocks
- Set query timeouts
- Use connection pool limits with appropriate max/min
- Log slow queries above a threshold

**File system**: Handle disk full, permission denied, and race conditions
- Check available space before large writes
- Use atomic write patterns (write to temp, rename)
- Handle `ENOENT` and `EACCES` explicitly

### 3. Data Safety
Treat production data with care at every level:
- **Never log sensitive data**: passwords, tokens, PII, credit card numbers
- **Never trust input size**: enforce limits on request bodies, file uploads, query parameters
- **Never expose internals**: stack traces, database schemas, internal paths in error responses
- **Always validate at boundaries**: data from users, APIs, message queues, and even your own database (data can be corrupted)

### 4. Resource Awareness
Know the resource profile of your code:

| Resource | Production Concern |
|----------|--------------------|
| Memory | Unbounded caches, large object allocations in loops, memory leaks from unclosed resources |
| CPU | Synchronous heavy computation blocking the event loop, regex backtracking, JSON parsing of huge payloads |
| Connections | Database connection pool exhaustion, HTTP keep-alive limits, socket leaks |
| Disk | Log rotation, temp file cleanup, upload storage limits |
| External rate limits | Third-party API quotas, email sending limits, OAuth token refresh rates |

### 5. Operational Readiness Thinking
While writing code, consider operational needs:
- **Can this be debugged?** — Add structured logging at decision points
- **Can this be monitored?** — Expose metrics for request rate, error rate, latency
- **Can this be rolled back?** — Database migrations should be reversible
- **Can this be scaled?** — Avoid server-local state; use stateless design patterns
- **Can this be configured?** — Use environment variables for values that differ across environments

### 6. Graceful Degradation
Design systems that bend instead of break:
- Return cached/stale data when the live source is down (with a warning)
- Disable non-critical features rather than crashing the entire service
- Queue work for later when downstream systems are temporarily unavailable
- Shed load intentionally (rate limiting) rather than collapsing under pressure

## Behavioral Rules

1. **Every external call needs a timeout** — Infinite waits are production incidents
2. **Every loop needs a bound** — Unbounded iteration on user-controlled data is a DoS vector
3. **Every error needs handling** — Unhandled exceptions crash processes
4. **Every log statement needs review** — Ask "would I want this in a compliance audit?"
5. **Every feature needs a kill switch** — Feature flags allow disabling code without deploying

## Failure Modes

| Failure | Symptom | Correction |
|---------|---------|------------|
| "Works on my machine" thinking | Code fails under production load, data volume, or network conditions | Test with realistic data sizes and failure scenarios |
| No timeouts on external calls | Requests hang indefinitely; thread/connection pool exhausts | Add timeouts to every network call, database query, and file operation |
| Logging sensitive data | Compliance violation discovered in production logs | Audit every log statement for PII, tokens, and credentials |
| Unbounded resource usage | Memory leaks, disk fills up, connection pools exhaust | Set explicit limits on caches, queues, pools, and temporary storage |
| No fallback behavior | One dependency failure cascades into full system outage | Implement graceful degradation for every external dependency |
