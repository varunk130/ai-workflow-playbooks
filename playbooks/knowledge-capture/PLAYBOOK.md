---
name: knowledge-capture
trigger: When making an architectural decision, creating an API, or onboarding a new team member
---

# Knowledge Capture

ADRs, API docs, and living documentation that outlast individual memory.

## Objective

Ensure critical knowledge is recorded where people actually look for it -- next to the
code, in searchable formats, with clear ownership. Document the **why** behind decisions
so future maintainers can evaluate whether the context still holds, rather than
re-litigating choices or blindly preserving them.

## Triggers

- An architectural or design decision is made (or revisited).
- A new API endpoint, library, or service is introduced.
- A README becomes stale or misleading.
- A production incident reveals undocumented operational procedures.
- A new team member asks a question whose answer should be permanent.

## Workflow

### 1. Record Architecture Decision Records (ADRs)

Create an ADR for every decision that shapes the system's structure, technology
choices, or cross-cutting concerns.

```
docs/adr/NNN-<slug>.md

# NNN. <Title>

**Status:** Proposed | Accepted | Deprecated | Superseded by NNN

## Context
What forces are at play? What problem are we solving?

## Decision
What did we decide and why?

## Consequences
What trade-offs does this create? What becomes easier or harder?
```

- Number ADRs sequentially; never reuse numbers.
- Mark superseded ADRs with a pointer to the replacement.
- Review ADRs during tech-debt triage to prune outdated decisions.

### 2. Maintain API Documentation

- Use **OpenAPI / Swagger** for REST APIs; generate specs from code annotations
  where possible to avoid drift.
- Include request/response examples, error codes, authentication requirements,
  and rate limits.
- For internal libraries, write **inline doc comments** on every public symbol.
- Publish docs to a discoverable location (developer portal, repo wiki, or
  generated site).

### 3. Keep READMEs Accurate

- Every repository has a root `README.md` covering: purpose, quickstart,
  prerequisites, configuration, and contributing guidelines.
- Update the README in the **same PR** that changes the behavior it describes.
- If a section grows beyond a few paragraphs, extract it into a dedicated doc
  and link from the README.

### 4. Document the Why, Not the What

- Code shows **what** happens; comments and docs explain **why**.
- Avoid comments that restate the code (`// increment counter`).
- Focus on intent, constraints, edge cases, and non-obvious trade-offs.

### 5. Write Operational Runbooks

- For every service, maintain a runbook covering: health checks, common failure
  modes, escalation paths, and recovery procedures.
- Store runbooks alongside the service code (`docs/runbook.md`) or in the
  team's operational wiki.
- Validate runbooks during incident retrospectives; update when procedures change.

## Guardrails

- **ADRs are required** for decisions that affect more than one service or team.
- **API docs are generated from source** where tooling supports it, to prevent drift.
- **README accuracy is a review checkpoint** -- reviewers verify docs match the change.
- **Runbooks are tested** at least once per quarter through tabletop exercises or
  game days.
- **No tribal knowledge.** If the answer is only in someone's head, it is not documented.

## Exit Criteria

- [ ] An ADR exists for every significant architectural decision.
- [ ] API documentation is published, current, and includes examples.
- [ ] The repository README accurately reflects setup and usage.
- [ ] Inline comments explain why, not what.
- [ ] Operational runbooks exist for every production service.
- [ ] Documentation is updated in the same PR as the code change it describes.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Alternative |
|---|---|---|
| Decision made in Slack, never recorded | Context lost when thread scrolls away | Write an ADR and link it from the thread |
| API docs written once, never updated | Consumers follow stale instructions, hit errors | Generate from source; enforce in CI |
| README that describes v1 of a v3 system | Misleads new developers, wastes onboarding time | Update README in the same PR as the change |
| Comments that restate the code | Noise that drifts from reality | Document intent, constraints, and trade-offs |
| Runbooks that have never been tested | False confidence; fail when needed most | Validate quarterly via tabletop exercises |
