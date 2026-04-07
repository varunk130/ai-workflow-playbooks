---
name: discovery-and-ideation
trigger: Use when exploring a new problem, brainstorming solutions, or scoping a project before writing a specification.
---

# Discovery & Ideation

## Objective
Transform a vague idea or user need into a well-scoped problem statement with ranked solution candidates. This prevents the common failure mode of agents jumping directly into code without understanding what they're solving.

## Triggers
- A new feature request or product idea arrives
- The problem space is ambiguous or underexplored
- Multiple possible approaches exist and need evaluation
- The user says "I want to build..." without detailed requirements

## Do Not Use When
- Requirements are already documented in a spec
- The task is a bug fix with clear reproduction steps
- You're iterating on an existing, well-defined feature

## Workflow

### Step 1: Capture the Raw Signal
Document the initial input exactly as received. Record:
- **Who** is requesting this (user, stakeholder, system)
- **What** they described (verbatim, no interpretation yet)
- **Why** they believe it's needed (stated motivation)
- **Where** it fits (existing system, greenfield, integration point)

### Step 2: Map the Problem Space
Before thinking about solutions, understand the problem:
1. Identify the core user pain or unmet need
2. List the constraints (technical, timeline, budget, compatibility)
3. Document existing solutions or workarounds users employ today
4. Define what "solved" looks like from the user's perspective

### Step 3: Diverge — Generate Candidates
Produce at least 3 distinct solution approaches:
- **Approach A**: The simplest thing that could work
- **Approach B**: The most complete solution
- **Approach C**: A creative alternative (different technology, different framing, different scope)

For each approach, note:
- Effort estimate (T-shirt size: S/M/L/XL)
- Key technical risks
- What it does NOT solve

### Step 4: Converge — Evaluate and Rank
Score each candidate against:

| Criterion | Weight | Approach A | Approach B | Approach C |
|-----------|--------|------------|------------|------------|
| Solves core need | High | | | |
| Implementation effort | Medium | | | |
| Maintenance burden | Medium | | | |
| Risk level | High | | | |
| Extensibility | Low | | | |

Recommend the highest-scoring approach with a one-paragraph justification.

### Step 5: Produce the Discovery Brief
Output a structured brief:
```
## Discovery Brief
### Problem Statement: [One sentence]
### Core User Need: [What the user actually needs]
### Recommended Approach: [Name and summary]
### Key Risks: [Top 3]
### Open Questions: [Things still unclear]
### Next Step: Proceed to Specification Authoring
```

## Guardrails
- Never skip divergence. Generating only one approach is planning, not discovery
- Do not evaluate approaches before generating all candidates (premature convergence)
- Assumptions must be stated explicitly — never silently assumed
- If open questions remain, document them rather than guessing answers

## Exit Criteria
- [ ] Problem statement is documented in one clear sentence
- [ ] At least 3 solution candidates were generated and evaluated
- [ ] A recommended approach is selected with justification
- [ ] Key risks and open questions are listed
- [ ] Discovery brief is produced and ready for specification

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| "The solution is obvious, let's just build it" | Skipping discovery leads to solving the wrong problem |
| "We only need one approach" | Single-candidate evaluation has no comparison baseline |
| Assuming constraints instead of asking | Hidden assumptions surface as blockers mid-implementation |
| Jumping to technology choices first | Technology is a means, not the starting point |
