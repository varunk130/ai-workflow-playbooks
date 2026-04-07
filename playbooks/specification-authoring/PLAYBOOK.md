---
name: specification-authoring
trigger: Use when defining requirements before implementation. Activate after discovery or when a feature needs a formal spec.
---

# Specification Authoring

## Objective
Produce a complete, unambiguous specification that serves as the single source of truth for implementation. A good spec eliminates the "what should this do?" questions that derail agents mid-build.

## Triggers
- After completing Discovery & Ideation
- Before any implementation work begins
- When requirements exist informally (in chat, tickets, verbal descriptions)
- When multiple agents or team members will work on the same feature

## Do Not Use When
- Fixing a clearly defined, isolated bug
- Making a cosmetic or copy change
- The spec already exists and is current

## Workflow

### Step 1: Gather Inputs
Collect all available context:
- Discovery brief (if available)
- User stories, tickets, or feature requests
- Existing system documentation
- Relevant API contracts or data schemas
- Constraints (performance budgets, compatibility requirements, deadlines)

### Step 2: Draft the Specification
Write a structured document covering these sections:

#### 2a. Purpose & Motivation
- What problem does this solve?
- Who benefits and how?
- What is the expected outcome when this ships?

#### 2b. Functional Requirements
- Enumerate each behavior the system must exhibit
- Use the format: "The system SHALL [verb] [object] when [condition]"
- Mark requirements as MUST, SHOULD, or MAY (RFC 2119 style)

#### 2c. Non-Functional Requirements
- Performance targets (response time, throughput)
- Availability and reliability expectations
- Security requirements (authentication, authorization, data handling)
- Accessibility standards (WCAG level)

#### 2d. System Boundaries
- What is IN scope for this spec
- What is explicitly OUT of scope
- Integration points with other systems

#### 2e. Data Model
- Key entities and their relationships
- Required fields, types, and constraints
- State transitions (if applicable)

#### 2f. User-Facing Behavior
- Describe each user interaction flow
- Include happy path, error states, and edge cases
- Define exact error messages and status codes

#### 2g. Testing Strategy
- Which requirements need unit tests
- Which flows need integration tests
- Which user journeys need end-to-end tests
- Acceptance criteria for each requirement

### Step 3: List Assumptions
Explicitly state every assumption. Format:
```
ASSUMPTION: [statement]
RISK IF WRONG: [what breaks]
VALIDATION: [how to verify this assumption]
```

### Step 4: Identify Open Questions
List anything unresolved. Each item should include:
- The question
- Who can answer it
- What the default assumption is if no answer arrives
- Whether it blocks implementation

### Step 5: Review the Spec
Before considering the spec complete:
1. Read every functional requirement — can you write a test for it?
2. Read every boundary — is there ambiguity about what's in or out?
3. Check for contradictions between requirements
4. Verify all error states are covered

## Guardrails
- Never start implementation with an incomplete spec. If sections are missing, document them as TODOs with owners
- Specifications are living documents — update them when requirements change during implementation
- Avoid implementation details in the spec. Describe WHAT, not HOW
- Every requirement must be testable. If you can't write a test for it, it's not specific enough

## Exit Criteria
- [ ] All seven spec sections are completed (or marked N/A with justification)
- [ ] Every functional requirement uses SHALL/SHOULD/MAY language
- [ ] Assumptions are listed with risk assessments
- [ ] Open questions are documented with default assumptions
- [ ] A peer (human or guardian) has reviewed the spec
- [ ] The spec is committed to the repository

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| "Let's figure it out as we go" | Ambiguity surfaces as rework, not progress |
| Writing implementation details into the spec | Couples the spec to a solution, reducing flexibility |
| Skipping non-functional requirements | Performance and security become afterthoughts |
| Spec with no testing strategy | No way to verify the implementation matches the spec |
| "Everyone knows what we mean" | Implicit knowledge doesn't transfer to AI agents |
