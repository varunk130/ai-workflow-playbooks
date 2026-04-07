---
name: complexity-reduction
trigger: when code is difficult to understand, functions exceed complexity thresholds, or dead code accumulates
---

# Complexity Reduction

Simplifying code while preserving behavior. Every simplification must be
backed by tests that prove the before and after are equivalent.

## Objective

Reduce the cognitive and structural complexity of a codebase by eliminating
dead code, decomposing oversized functions, simplifying conditional logic, and
inlining unnecessary abstractions -- all without changing observable behavior.

## Triggers

- A function's cyclomatic complexity exceeds 10.
- A function exceeds 500 lines (the Rule of 500).
- Dead code is detected by static analysis or coverage tools.
- A reviewer flags code as "hard to follow" or "over-abstracted."
- An agent encounters deeply nested logic that impedes reasoning.

## Workflow

### 1. Understand Before You Remove (The Fence Principle)

> If you come across a fence in the middle of a field, do not tear it down
> until you understand why it was put there.

- Read the history: `git log` and `git blame` on the target code to understand
  its origin, the problem it solved, and any edge cases it guards against.
- Read the tests that cover the code; if none exist, **write characterization
  tests first** that capture current behavior before making any changes.
- Ask: Is this code dead, or is it reachable through a path I have not
  considered?

### 2. Measure Complexity

- Use cyclomatic complexity as a guide. Aim for **fewer than 10 paths per
  function**. Tools: ESLint `complexity` rule, `radon` (Python), `gocyclo`.
- Count nesting depth -- any function with more than 3 levels of indentation
  deserves scrutiny.
- Check function length. The **Rule of 500**: any function exceeding 500 lines
  is a decomposition candidate, no exceptions.

### 3. Eliminate Dead Code

- Identify dead code through static analysis, coverage reports, and manual
  trace of call sites.
- **Confidence checks** before deletion:
  - Search the entire codebase for references (including strings, config
    files, and dynamic invocations).
  - Verify that the code is not toggled by a feature flag or environment
    variable.
  - Check for reflection-based or convention-based invocation (e.g., route
    handlers discovered by naming convention).
- Delete with a clear commit message explaining why the code is dead.

### 4. Simplify Conditional Logic

- **Early returns and guard clauses**: Replace deeply nested if/else trees
  with early exits for exceptional or boundary cases.
- **Consolidate conditions**: Merge adjacent `if` blocks with the same body;
  extract complex boolean expressions into named variables or functions.
- **Replace conditionals with polymorphism** when a switch or if-chain
  dispatches on type -- but only when there are 3+ branches and the types
  are stable.
- **Remove double negatives**: `if (!isNotValid)` becomes `if (isValid)`.

### 5. Inline Over-Abstracted Helpers

- If a helper function is called exactly once and its name merely restates
  its single-line body, inline it.
- If a wrapper adds no logic, no error handling, and no name clarity, remove
  the indirection.
- Preserve abstractions that genuinely encapsulate a concept, hide a complex
  implementation, or are used across multiple call sites.

### 6. Decompose Oversized Functions

- Extract cohesive blocks into well-named functions. Each extracted function
  should have a single clear responsibility.
- Prefer pure functions (input in, output out, no side effects) for extracted
  pieces -- they are easier to test and reason about.
- After decomposition, the parent function should read like a high-level
  outline of the operation.

## Guardrails

| Rule | Detail |
|------|--------|
| Tests first | Never simplify code that lacks test coverage; write characterization tests before changing. |
| Behavioral equivalence | Every simplification must preserve observable behavior; run the full test suite after each step. |
| Understand before removing | Apply the Fence Principle: know why the code exists before deleting it. |
| One transformation at a time | Make each simplification in its own commit so it can be reviewed and reverted independently. |
| Measure after | Re-check complexity metrics after changes to confirm improvement. |

## Exit Criteria

- All targeted functions have cyclomatic complexity below 10.
- No function exceeds 500 lines.
- Identified dead code has been removed with documented justification.
- Conditional logic uses guard clauses and early returns where applicable.
- Over-abstracted single-use helpers have been inlined.
- The full test suite passes with no behavioral changes.
- Complexity metrics show measurable improvement versus the starting baseline.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Approach |
|--------------|-------------|-----------------|
| Deleting code you do not understand | Removes safeguards for edge cases you have not considered. | Apply the Fence Principle; read history and tests first. |
| Simplifying without tests | No safety net means silent behavioral changes go undetected. | Write characterization tests before any refactor. |
| Replacing complexity with cleverness | A "simpler" one-liner that nobody can read is not simpler. | Optimize for readability, not line count. |
| Big-bang refactors | Changing everything at once makes failures hard to diagnose. | One transformation per commit; verify after each step. |
| Inlining useful abstractions | Removing a helper that encapsulates a genuine concept spreads duplication. | Only inline single-use helpers that add no clarity. |
| Ignoring dynamic references | Deleting "dead" code that is actually invoked via reflection or config. | Search strings, configs, and dynamic call sites before deletion. |
