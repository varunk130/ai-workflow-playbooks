---
name: incremental-verification
capability: Continuously verify work at every step rather than waiting until the end to test.
---

# Incremental Verification

## What This Skill Enables
An agent that confirms correctness at every micro-step — after every function, every file change, every integration point — rather than building a large body of code and hoping it works. Without this skill, agents produce code that looks plausible but fails at runtime, and bugs compound silently until the system is too broken to debug efficiently.

## Core Competencies

### 1. The Verify-As-You-Go Loop
After every meaningful change, run a verification:

```
Write code → Verify → Commit → Next change
     ↑                            |
     └────────────────────────────┘
```

Verification methods by change type:

| Change | Verification |
|--------|-------------|
| New function | Run its unit test |
| Modified function | Run existing tests + new test for the change |
| New API endpoint | Send a test request, check response shape and status |
| UI component | Render it, check the DOM/accessibility tree |
| Configuration change | Restart the service, verify it loads without errors |
| Database migration | Run the migration, verify the schema matches expectations |

### 2. Build Verification
Before writing new code, confirm the current state is clean:
- Run the full test suite — all green? Proceed
- Run the build command — compiles without errors? Proceed
- Run the linter — no new violations? Proceed

If any of these fail before you've made changes, flag it immediately. Do not write code on top of a broken foundation.

### 3. Assertion Density
Write assertions liberally during development:
- After fetching data: assert it has the expected shape
- After transforming data: assert the output matches expectations
- After calling an external service: assert the response status is successful
- After state transitions: assert the new state is valid

These can be formal tests or temporary debug assertions that get removed later — the point is catching errors at the moment they occur, not ten steps later.

### 4. Regression Detection
After every change, verify nothing broke:

```
CHANGED: src/auth/validate.ts

VERIFICATION STEPS:
1. Run unit tests for auth module → ✅ 12/12 pass
2. Run integration tests touching auth → ✅ 5/5 pass
3. Run full test suite → ✅ 147/147 pass
4. No new linter warnings → ✅

RESULT: Change verified. Committing.
```

If a test fails that you didn't expect, stop. Don't fix it silently. Understand why the change affected that test.

### 5. End-of-Task Verification
When a task (not just a single change) is complete:
1. Run the complete test suite
2. Verify the acceptance criteria from the task definition
3. Check for console errors, warnings, or deprecation notices
4. Review your own diff — does anything look unintended?
5. If the task has UI, visually verify the result

## Behavioral Rules

1. **Never stack unverified changes** — Verify change A before starting change B
2. **Test failures are stop signals** — Don't proceed past a red test; fix it first
3. **The build must always be green** — A broken build is the highest-priority fix
4. **Unexpected passes are suspicious** — A test that passes when you expected it to fail needs investigation
5. **Commit verified work frequently** — Every green state is a checkpoint worth saving

## Failure Modes

| Failure | Symptom | Correction |
|---------|---------|------------|
| Writing 500 lines then testing | Cascade of errors with unclear root cause | Verify after every function or small logical change |
| Ignoring test failures | Broken tests accumulate; suite becomes unreliable | Fix every failure immediately or investigate why |
| Only testing happy path | Code works for valid input, crashes on edge cases | Test error conditions and boundary values at each step |
| Skipping build verification | Runtime errors that the compiler would have caught | Run build after every file modification |
| Not checking your own diff | Accidental debug code, TODO comments, or unrelated changes ship | Review your diff before marking any task complete |
