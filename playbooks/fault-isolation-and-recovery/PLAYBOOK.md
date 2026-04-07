---
name: fault-isolation-and-recovery
trigger: when a bug is reported, a test fails unexpectedly, or system behavior deviates from specification
---

# Fault Isolation and Recovery

Systematic debugging from reproduction to resolution. Every fix must be
preceded by understanding, and followed by a guard that prevents recurrence.

## Objective

Given a defect or unexpected behavior, methodically isolate its root cause,
apply a minimal corrective change, and add protective measures so the same
class of fault cannot recur undetected.

## Triggers

- A bug report is filed (user-reported or automated alert).
- A previously passing test begins to fail.
- Observed behavior contradicts the specification or acceptance criteria.
- A deployment produces errors not present in staging.
- An agent encounters an unexpected runtime exception during a task.

## Workflow

### Step 1: Reproduce

- Establish a **reliable reproduction path** before touching any code.
- Document the exact inputs, environment, and sequence of actions.
- If the defect is intermittent, increase observation frequency or add
  temporary instrumentation (logging, counters) to capture it in the act.
- Reproduction in a local or isolated environment is mandatory; never debug
  solely against production.

### Step 2: Localize

- Read the error message **carefully and completely** before forming a
  hypothesis. Most error messages point directly at the fault location.
- Use binary search to narrow the fault scope:
  - **git bisect**: identify the exact commit that introduced the defect.
  - **Comment-out bisection**: disable halves of the suspect code path until
    the defect disappears, then re-enable to pinpoint the faulty section.
- Check recent changes first -- most defects live in code touched within the
  last few commits.
- Inspect boundaries: off-by-one errors, null/undefined inputs, type
  mismatches, and timezone or encoding differences.

### Step 3: Minimize

- Create a **minimal reproduction case** -- the smallest possible input and
  code path that still exhibits the fault.
- Strip away unrelated UI, data, and configuration until only the essential
  trigger remains.
- A minimal case accelerates understanding and becomes the seed for a
  regression test.

### Step 4: Fix

> **The stop-and-think rule**: Do not apply a fix until you can articulate,
> in one sentence, *why* the current code is wrong and *why* the proposed
> change corrects it.

- Apply the **smallest change** that addresses the root cause.
- Avoid shotgun fixes -- changing multiple things hoping one will work.
- If the root cause is unclear after reasonable effort, escalate rather than
  guess. A wrong fix is worse than no fix.
- Consider safe fallback patterns where appropriate:
  - **Try/catch boundaries** at integration seams to prevent cascading failures.
  - **Circuit breakers** for external service calls to fail fast under
    sustained errors.
  - **Default/fallback values** for non-critical data paths, with logging
    when the fallback activates.

### Step 5: Guard

- Write a **regression test** that fails without the fix and passes with it.
  This is non-negotiable -- every fix ships with a test.
- If the defect class is broad (e.g., missing null checks), consider adding a
  lint rule or static analysis check.
- Update documentation or runbooks if the fault involved operational
  misunderstanding.
- Review adjacent code for the same pattern -- the same mistake often exists
  in nearby functions.

## Guardrails

| Rule | Detail |
|------|--------|
| Reproduce before you fix | No fix without a reliable reproduction path. |
| Read the error message first | Resist the urge to guess; most messages are informative. |
| One hypothesis at a time | Test each theory in isolation; multi-variable changes obscure causation. |
| Stop and think before applying | Articulate the root cause in one sentence before writing the fix. |
| Every fix gets a regression test | No exceptions. The test must fail without the fix. |
| Prefer minimal changes | Scope the fix to the root cause; avoid opportunistic refactoring in the same commit. |
| Escalate when stuck | If 30 minutes of focused investigation yields no progress, seek a second pair of eyes. |

## Exit Criteria

- The defect is reliably reproduced and the reproduction is documented.
- Root cause is identified and articulated in a clear one-sentence explanation.
- A minimal fix is applied that addresses only the root cause.
- A regression test exists that fails without the fix and passes with it.
- Adjacent code has been scanned for the same fault pattern.
- No new console errors, test failures, or behavioral regressions are
  introduced by the fix.
- If fallback patterns were added, they include logging for observability.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Approach |
|--------------|-------------|-----------------|
| Fix-first, understand-later | Applying changes without knowing the cause leads to masking bugs or introducing new ones. | Always articulate the root cause before writing a fix. |
| Shotgun debugging | Changing multiple things at once makes it impossible to know what actually fixed it. | Test one hypothesis at a time in isolation. |
| Skipping the regression test | Without a test, the same bug will return in a future refactor. | Write the failing test first, then apply the fix. |
| Debugging in production only | Production has noise, concurrency, and limited tooling that obscure root causes. | Reproduce locally; use production only for data gathering. |
| Blaming the framework | Assuming the bug is in a dependency before checking your own code wastes time. | Verify your code is correct first; file upstream issues with minimal reproductions. |
| Fixing symptoms instead of causes | Wrapping a null check around a crash site without asking why the value is null. | Trace the data flow back to its origin to find the real fault. |
| Over-engineering the guard | Building an elaborate safety net for a one-off typo. | Match the guard's complexity to the defect's likelihood and severity. |
