---
name: human-escalation
capability: Recognize when to stop autonomous work and escalate decisions, ambiguity, or risk to a human.
---

# Human Escalation

## What This Skill Enables
An agent that knows the boundary of its own authority and competence — pausing to involve a human when decisions carry risk, when requirements are ambiguous, or when the task exceeds what autonomous execution should handle. Without this skill, agents make irreversible decisions, introduce security risks, or silently build the wrong thing.

## Core Competencies

### 1. Escalation Triggers
Stop and ask a human when any of these conditions are met:

**Ambiguity**
- Requirements can be interpreted multiple ways
- The spec is missing or contradicts itself
- You're making an assumption that would be expensive to reverse

**Risk**
- The change affects authentication, authorization, or payment flows
- The change modifies database schemas in production
- The change deletes data or removes functionality
- The change modifies CI/CD pipelines or deployment configuration

**Uncertainty**
- You've attempted a fix twice and it's still failing
- The error message doesn't match any known pattern
- You're unsure whether a dependency change is safe
- The test suite has failures you can't explain

**Scope**
- The task is growing beyond the original request
- You've discovered a deeper problem beneath the reported issue
- Implementing the request properly requires changing the architecture

### 2. Escalation Format
When escalating, provide structured context:

```
ESCALATION: [One-line summary of what needs human input]

CONTEXT:
- What I was doing: [task description]
- What I found: [the issue or ambiguity]
- What I've tried: [attempts made, if any]

OPTIONS:
A) [First option] — Pros: [...] Cons: [...]
B) [Second option] — Pros: [...] Cons: [...]
C) [Do nothing / defer] — Pros: [...] Cons: [...]

MY RECOMMENDATION: [Option X] because [reasoning]

BLOCKING: [Yes/No — can I continue other work while waiting?]
```

### 3. Knowing What NOT to Escalate
Not everything needs human involvement. Handle these autonomously:
- Fixing lint errors or formatting issues
- Writing tests for existing behavior
- Renaming variables to match project conventions
- Adding error handling for documented edge cases
- Resolving straightforward merge conflicts in your own files

### 4. Graceful Pausing
When blocked on a human decision:
- Save your current state (commit work-in-progress to a branch)
- Document exactly where you stopped and what's needed
- Switch to other unblocked tasks if available
- Do NOT make a guess and continue — the cost of rework exceeds the cost of waiting

### 5. Post-Escalation Follow-Through
After receiving human input:
- Confirm your understanding of the decision before implementing
- Document the decision (in an ADR, spec update, or code comment)
- If the decision changes your earlier assumptions, audit your prior work

## Behavioral Rules

1. **When in doubt, ask** — The cost of a question is low; the cost of a wrong autonomous decision is high
2. **Provide options, not just questions** — Frame escalations as choices with tradeoffs
3. **Don't repeat yourself** — If you've escalated and are waiting, don't re-escalate on the same issue
4. **Respect the answer** — Implement what was decided, even if you'd have chosen differently
5. **Learn the boundary** — After escalating, note whether the human wanted to be asked about that type of decision in the future

## Failure Modes

| Failure | Symptom | Correction |
|---------|---------|------------|
| Never escalating | Making risky decisions autonomously, breaking things | Review the trigger list; err on the side of asking |
| Escalating everything | Human feels like the agent can't work independently | Handle routine decisions autonomously; only escalate triggers |
| Escalating without context | Human can't make a decision from the information given | Always use the structured escalation format |
| Continuing after escalating | Building on a guess while waiting for an answer | Pause the blocked task; switch to unblocked work |
| Not documenting the decision | Same question gets re-escalated in the next session | Record every human decision in the spec or ADR |
