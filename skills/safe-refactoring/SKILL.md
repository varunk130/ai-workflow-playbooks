---
name: safe-refactoring
capability: Restructure code without changing behavior, verified by tests at every step.
---

# Safe Refactoring

## What This Skill Enables

Agents with this skill can improve code structure -- readability, modularity, performance -- while guaranteeing that external behavior stays identical. Without it, agents tend to rewrite large sections at once, introduce subtle regressions, and produce diffs that are impossible for humans to review confidently. Safe refactoring turns a risky "big bang" rewrite into a chain of small, provably correct transformations.

## Core Competencies

### 1. Decide Whether to Refactor at All

Not every piece of ugly code needs attention right now. Before touching anything, answer these questions:

| Question | If "No", stop here |
|---|---|
| Is this code on the path of the current task? | Do not refactor drive-by style. |
| Do passing tests already cover the behavior? | Write tests first (see competency 2). |
| Is the improvement worth the review cost? | A rename across 200 files is noisy for little gain. |
| Is there a deadline within hours? | Ship first, refactor in a follow-up branch. |

Refactoring that is not motivated by a concrete upcoming change is speculative. Avoid it.

### 2. Lock Behavior with Tests Before Changing Structure

The "lock then change" pattern is non-negotiable:

```
Step 1 -- Write or confirm tests that cover the code you are about to move.
Step 2 -- Run the full relevant test suite. It must pass.
Step 3 -- Perform ONE refactoring move (see competency 3).
Step 4 -- Run the suite again. It must still pass.
Step 5 -- Commit with a single-purpose message.
Step 6 -- Repeat from Step 3 for the next move.
```

If no tests exist and writing them is impractical, add at minimum a characterization test: call the function with known inputs and assert on the current outputs, even if those outputs look wrong. The point is to detect change, not to validate correctness.

### 3. Use Named Refactoring Moves

Each commit should contain exactly one kind of transformation. Mixing moves makes regressions hard to bisect.

| Move | When to Use | Key Risk |
|---|---|---|
| **Extract Function** | A block inside a function has a clear name and purpose. | Getting the parameter list wrong; forgetting closures. |
| **Inline Function** | A wrapper adds no clarity and just indirects. | Callers may depend on the wrapper's name in tests or logs. |
| **Rename** | A name is misleading or does not match domain language. | Must update every reference, including strings and config. |
| **Move to Module** | A function lives in the wrong file or layer. | Import paths change; circular dependencies may surface. |
| **Simplify Conditional** | Nested if/else can be replaced with guard clauses or a lookup table. | Subtle short-circuit logic may be intentional. |

### 4. Keep Commits Single-Purpose

A refactoring commit must never include behavioral changes. If you realize a bug while refactoring, stash the refactoring work, fix the bug in its own commit with its own test, then resume. This keeps `git bisect` useful and code review fast.

### 5. Verify No Behavioral Change

After each move, run:

1. The unit tests covering the changed code.
2. Any integration tests that exercise the changed module.
3. A type check or linter pass if the language supports it.

If the project has snapshot tests, expect them to remain identical. If a snapshot changes, the refactoring altered behavior -- revert and investigate.

## Behavioral Rules

1. Never combine a refactoring commit with a feature or bug-fix commit.
2. Never refactor code that lacks test coverage without adding coverage first.
3. Never perform more than one named refactoring move per commit.
4. Always run the test suite after every individual move, not just at the end.
5. If tests fail after a move, revert the move immediately rather than attempting to fix forward.

## Failure Modes

| Failure | Symptom | Correction |
|---|---|---|
| Big-bang rewrite | Single commit touches dozens of files with mixed intent. | Break into one-move-per-commit sequence. |
| Refactoring without tests | "It looks the same" is offered instead of proof. | Write characterization tests before starting. |
| Drive-by refactoring | Unrelated files are cleaned up while working on a feature. | Limit scope to files on the current task's path. |
| Rename avalanche | A rename propagates across the entire codebase. | Assess review cost first; use IDE tooling; split across commits. |
| Fix smuggled in refactor | A bug fix hides inside a "refactoring" commit. | Separate the fix into its own commit with its own test. |
