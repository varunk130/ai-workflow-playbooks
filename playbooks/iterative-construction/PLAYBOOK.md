---
name: iterative-construction
trigger: Use when implementing features. Activate during the build phase to ensure incremental, verifiable progress.
---

# Iterative Construction

## Objective
Build software in thin vertical slices where each slice delivers a working increment that can be tested, demonstrated, and rolled back independently. Avoid the "big bang" anti-pattern where nothing works until everything works.

## Triggers
- Task decomposition is complete and you're ready to implement
- Building a feature that spans multiple components
- Implementing changes that need to coexist with existing functionality

## Workflow

### Step 1: Select the Next Task
From the implementation sequence:
1. Pick the next unblocked task (all dependencies satisfied)
2. Read its acceptance criteria
3. Verify prerequisites exist (required files, APIs, test infrastructure)

### Step 2: Prepare the Work Surface
Before writing feature code:
1. Ensure the current codebase builds and passes all tests: `npm test` (or equivalent)
2. Create a checkpoint (commit or branch) so you can revert cleanly
3. Read the relevant existing code — understand what you're changing

### Step 3: Implement in Micro-Increments
Follow this inner loop for each task:

```
┌─────────────────────────────────────┐
│  1. Write the smallest useful change│
│  2. Run tests → must pass           │
│  3. Verify behavior manually if UI  │
│  4. Commit with descriptive message │
│  5. Repeat until task is complete   │
└─────────────────────────────────────┘
```

Rules for each increment:
- **One concern per commit**: Don't mix refactoring with feature work
- **Tests travel with code**: If you add a function, add its test in the same commit
- **Safe defaults**: New code paths should be disabled by default (feature flags, config toggles)
- **No broken windows**: If tests fail after your change, fix them before proceeding

### Step 4: Wire the Vertical Slice
A vertical slice connects all layers for one user-visible behavior:
- Data model → business logic → API endpoint → UI component
- Each slice should be demonstrable: "User can now [action] and see [result]"

### Step 5: Validate the Task
Run acceptance criteria from the task definition:
1. All specified tests pass
2. Manual verification of the behavior (if applicable)
3. No regressions in existing functionality
4. Code follows project conventions

### Step 6: Mark Complete and Move On
- Update the task status
- Note any deviations from the plan (scope changes, discovered requirements)
- Select the next task (return to Step 1)

## Guardrails
- Never push code that breaks existing tests. The mainline must always be green
- Do not implement tasks out of dependency order unless you've verified no coupling exists
- If a task grows beyond its estimated size, stop and re-decompose rather than pushing through
- Feature flags are required for user-visible changes that aren't yet complete
- Refactoring is a separate task — do not refactor while implementing features

## Exit Criteria
- [ ] Each implemented task passes its acceptance criteria
- [ ] No existing tests are broken
- [ ] Each vertical slice is independently demonstrable
- [ ] Commits are atomic and descriptive
- [ ] Feature flags protect incomplete functionality

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| "Let me build the whole thing first, then test" | Integration bugs hide until the end, compounding fix cost |
| Mixing refactoring with feature work | Impossible to isolate which change caused a regression |
| Skipping the checkpoint/commit step | No rollback point when things go wrong |
| Implementing the UI before the API exists | Creates mocking overhead and integration risk |
| "I'll add the feature flag later" | Incomplete features ship to users accidentally |
