---
name: peer-review-protocol
trigger: when code changes are ready for review, a pull request is opened, or an agent is asked to evaluate a diff
---

# Peer Review Protocol

Multi-axis code review that produces structured, actionable verdicts. Good
review catches defects early, raises the quality bar, and spreads knowledge
across the team.

## Objective

Evaluate a code change across five axes -- correctness, readability, architecture
fitness, security surface, and performance impact -- and deliver a clear verdict
with categorized findings that the author can act on without ambiguity.

## Triggers

- A pull request is opened or updated and review is requested.
- An agent is asked to review a diff, a file, or a set of changes.
- A pre-merge quality gate requires structured sign-off.
- A team member requests a second opinion on an implementation approach.

## Workflow

### 1. Prepare: Read the Spec First

- Before reading any code, read the linked issue, ticket, or specification.
- Understand the **intent** of the change -- what problem it solves and what
  constraints it operates under.
- If there is no spec, ask the author for a one-paragraph summary before
  proceeding.

### 2. Review Tests Before Implementation

- Open the test files first. Tests reveal the author's understanding of
  requirements and edge cases.
- Check for:
  - Coverage of happy path, error path, and boundary conditions.
  - Clarity of test names (can you understand the requirement from the name?).
  - Absence of tests for critical paths (flag as a Critical finding).

### 3. Evaluate Across Five Axes

#### Correctness
- Does the code do what the spec requires?
- Are edge cases handled (null, empty, overflow, concurrent access)?
- Do types and contracts match across boundaries?

#### Readability
- Can a new team member understand this code without the author explaining it?
- Are names descriptive and consistent with project conventions?
- Is control flow straightforward, or does it require mental gymnastics?

#### Architecture Fitness
- Does the change fit the existing architecture, or does it introduce a new
  pattern without justification?
- Are responsibilities in the right layers (UI vs. business logic vs. data)?
- Does it respect module boundaries and dependency direction?

#### Security Surface
- Are user inputs validated and sanitized?
- Are secrets, tokens, or credentials kept out of code and logs?
- Does the change introduce new attack vectors (injection, CSRF, open
  redirects, insecure deserialization)?

#### Performance Impact
- Does the change introduce O(n^2) or worse algorithmic complexity?
- Are there unnecessary re-renders, redundant queries, or missing caches?
- Will this scale to 10x the current data volume?

### 4. Classify Findings by Severity

| Severity | Meaning | Author Action |
|----------|---------|---------------|
| **Critical** | Defect, security hole, or data-loss risk. Blocks merge. | Must fix before merge. |
| **Major** | Significant quality concern. Should not ship as-is. | Should fix; justify deferral explicitly if not. |
| **Minor** | Style, naming, or minor improvement suggestion. | Consider; no merge block. |

### 5. Acknowledge Good Practices

- Explicitly call out things done well: clean abstractions, thorough tests,
  good documentation, clever-but-readable solutions.
- Positive feedback reinforces good patterns and makes review a balanced
  conversation, not just a list of complaints.

### 6. Deliver Structured Output

Produce the review in the following format:

```
## Review: <PR title or change description>

**Verdict**: APPROVE | REQUEST_CHANGES

### Critical
- [ ] <file:line> -- <description of issue and suggested fix>

### Major
- [ ] <file:line> -- <description of issue and suggested fix>

### Minor
- [ ] <file:line> -- <description of suggestion>

### Positive
- <file:line> -- <what was done well and why it matters>

### Summary
<1-3 sentence overall assessment>
```

### 7. Change Sizing Norms

- A single review unit should target roughly **100 lines of logical change**
  (excluding generated code, lock files, and large test fixtures).
- Changes above 300 lines should be split into stacked PRs or reviewed in
  focused passes (tests first, then core logic, then wiring).
- If the diff is too large to hold in working memory, say so and request a
  split.

## Guardrails

| Rule | Detail |
|------|--------|
| Spec before code | Always read the spec/ticket before the diff. |
| Tests before implementation | Review test coverage and quality first. |
| One pass per axis | Do not try to evaluate all five axes simultaneously. |
| Severity must be explicit | Every finding must carry a severity label. |
| Acknowledge the good | Every review must include at least one positive observation. |
| No style nitpicks on Critical PRs | If there are Critical findings, defer Minor style comments to a follow-up. |
| Timebox reviews | Spend no more than 60 minutes on a single review pass; request a split if needed. |

## Exit Criteria

- All five review axes have been evaluated.
- Every finding is classified as Critical, Major, or Minor.
- A clear verdict (APPROVE or REQUEST_CHANGES) is stated.
- At least one positive practice is acknowledged.
- The structured output format is used so findings are actionable.
- For REQUEST_CHANGES verdicts, every Critical and Major item includes a
  concrete suggestion or pointer toward resolution.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Approach |
|--------------|-------------|-----------------|
| Rubber-stamp approval | Misses defects and erodes trust in the review process. | Evaluate every axis; if time-constrained, say which axes you skipped. |
| Style-only reviews | Ignores correctness and security while fixating on formatting. | Automate style enforcement with linters; focus human review on logic. |
| Review without reading the spec | Evaluating code without knowing the intent leads to irrelevant feedback. | Always read the spec or ask for a summary first. |
| Unclassified comments | The author cannot prioritize when everything is a vague suggestion. | Assign a severity to every finding. |
| Reviewing giant diffs in one pass | Cognitive overload causes reviewers to miss important issues. | Request splits or review in focused passes by concern. |
| Only negative feedback | Demoralizes authors and discourages good practices. | Balance critique with explicit positive observations. |
| Blocking on Minor issues | Holding up a merge for trivial preferences slows the team. | Approve with Minor suggestions noted for follow-up. |
