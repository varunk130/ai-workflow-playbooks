---
name: test-first-engineering
trigger: Use when building or modifying features that deliver user value. Activate before writing implementation code to ensure every behavior is proven by a test.
---

# Test-First Engineering

## Objective
Write tests before implementation so that every feature is proven to work by an automated check that existed before the code did. Tests are not afterthoughts bolted on for coverage metrics — they are the specification expressed in executable form. If a feature has value, it must have a test that proves that value.

## Triggers
- About to implement a new feature, endpoint, component, or utility
- Fixing a bug (reproduce it with a failing test first)
- Refactoring existing code (lock behavior with tests before changing structure)
- Adding validation rules, business logic, or state transitions
- Integrating with an external system or API boundary

## Do Not Use When
- Writing throwaway prototypes or exploratory spikes (tag them as such and delete after)
- Modifying purely declarative configuration files (YAML, JSON configs) with no behavioral impact
- Generating boilerplate scaffolding that will be immediately covered by subsequent test-first work

## Workflow

### Step 1: Understand the Behavior Under Test
Before writing any test or implementation:
1. Read the requirement, acceptance criteria, or bug report
2. Express the desired behavior as one or more plain-language assertions:
   - "Given [context], when [action], then [outcome]"
3. Identify the **public contract** being tested — inputs, outputs, side effects
4. Determine the test size category (see Test Sizing below)

Do not write a test until you can state what it proves in one sentence.

### Step 2: Write the Failing Test (Red)
Create the test file (or add to the existing test module for the unit under test):

```
[test-file]
describe('[Unit Under Test]', () => {
  describe('[method or behavior]', () => {
    it('should [expected behavior in plain language]', () => {
      // Arrange — set up preconditions
      // Act — invoke the behavior
      // Assert — verify the outcome
    });
  });
});
```

Rules for this step:
- **Run the test and confirm it fails**. A test that passes before implementation proves nothing
- The failure message should clearly describe what is missing (not a cryptic stack trace)
- Write exactly one test. Do not batch-write multiple failing tests
- Name the test so a stranger can understand the requirement without reading the body:
  - Good: `it('rejects withdrawal when balance is insufficient')`
  - Bad: `it('test case 3')`

### Step 3: Write the Minimum Implementation (Green)
Write the smallest amount of production code that makes the failing test pass:
1. Implement only what the test demands — nothing speculative
2. Hard-code return values if that satisfies the test (the next test will force generalization)
3. Run the full test suite, not just the new test
4. If any existing test breaks, fix the regression before proceeding

```
┌────────────────────────────────────────────┐
│  FAILING TEST (Red)                        │
│       ↓                                    │
│  MINIMUM IMPLEMENTATION (Green)            │
│       ↓                                    │
│  ALL TESTS PASS?                           │
│     ├── No → Fix until green               │
│     └── Yes → Proceed to Refactor          │
└────────────────────────────────────────────┘
```

Resist the urge to "finish" the implementation. Let the tests pull the design forward.

### Step 4: Refactor Under Green Tests (Refactor)
With all tests passing, improve the code:
1. Remove duplication in production code
2. Improve naming and readability
3. Extract helper functions or modules if complexity warrants it
4. Remove duplication in test code (shared fixtures, helper builders)
5. Run all tests after each refactoring move — stay green

Refactoring rules:
- Do not add new behavior during refactoring (no new tests should be needed)
- Do not change the public contract — only internals
- If you discover a missing behavior, stop refactoring and return to Step 2

### Step 5: Repeat the Cycle
Return to Step 2 for the next behavior. Each cycle should take **5-15 minutes**. If a cycle exceeds 30 minutes, the behavior is too large — decompose it.

Progress rhythm:
```
Red → Green → Refactor → Red → Green → Refactor → ...
```

### Step 6: Evaluate Test Composition
After completing a feature's test suite, review the test distribution against the test pyramid:

```
        ╱  E2E   ╲        ~5% of tests
       ╱──────────╲       Critical user journeys only
      ╱ Integration ╲     ~20% of tests
     ╱────────────────╲   API boundaries, DB queries, service calls
    ╱     Unit Tests    ╲  ~75% of tests
   ╱────────────────────╲ Pure logic, transformations, validators
```

Ask these questions:
- Are there integration tests that should be unit tests? (testing pure logic through HTTP)
- Are there E2E tests that duplicate what integration tests already cover?
- Are there unit tests for trivial getters/setters that add noise without value?

## Test Sizing

Classify every test by its resource requirements, not by what it tests:

| Size | Constraints | Examples |
|------|------------|----------|
| **Small** | Single process, no I/O, no network, no disk, < 1s | Pure function logic, validators, transformers, state machines |
| **Medium** | May use localhost network or local filesystem, < 10s | Database queries against test containers, HTTP handlers with in-memory server |
| **Large** | May use external networks, multiple processes, < 60s | Full E2E browser flows, multi-service integration, tests requiring deployed infrastructure |

Prefer small tests. When a test requires medium or large size, ask: "Can I restructure the code to extract the pure logic into a small-testable unit?"

## Mocking Strategy

Mock at boundaries, not at internals:

```
✅ Mock the HTTP client that calls a third-party API
✅ Mock the database driver when testing business logic
✅ Mock the clock/timer for time-dependent behavior
✅ Mock the filesystem when testing backup rotation logic

❌ Mock a private helper function within the module under test
❌ Mock the return value of a sibling method on the same class
❌ Mock data transformations between internal layers
❌ Mock everything to make a test "fast" — restructure instead
```

Guidelines:
- **Own code should be called, not mocked**. If you mock your own function, the test proves nothing about your system
- **Boundary mocks should be thin**. A mock that reimplements business logic is a maintenance trap
- **Prefer fakes over mocks** where feasible: an in-memory repository is better than mocking 15 method calls on a repository interface
- **Verify behavior, not call sequences**. Assert on outcomes (return values, state changes, emitted events) rather than asserting that method X was called with arguments Y in order Z
- If your test has more mock setup than assertions, the design is coupling too tightly to implementation details

### When to Choose Integration Over Unit Tests
Write an integration test instead of (or in addition to) a unit test when:
- The behavior's value is **in the wiring** between components, not in isolated logic
- You are testing database queries (an in-memory fake diverges from real SQL behavior)
- You are testing HTTP middleware chains where ordering matters
- You are testing serialization/deserialization at API boundaries
- A unit test would require so many mocks that it tests nothing real

Write a unit test when:
- The logic is a pure function with clear inputs and outputs
- The behavior is algorithmic (sorting, filtering, calculation, validation)
- You need to cover many edge cases efficiently (parameterized/table-driven tests)
- The code has no side effects or dependencies on external state

## Guardrails
- Never commit code without the accompanying test. Tests and implementation travel together
- Never skip a failing test to "come back later." A skipped test is a lie about system behavior
- Never write a test that depends on the execution order of other tests
- Never share mutable state between test cases. Each test sets up its own world and tears it down
- If a test is flaky (passes sometimes, fails sometimes), treat it as a P0 bug — fix or delete immediately
- Do not optimize for code coverage percentage. Optimize for **behavior coverage**: every documented requirement has a corresponding test
- A test that never fails provides no information. If you cannot imagine a code change that would make the test fail, the test is trivial and should be removed
- Do not test framework or library internals. Test your code's interaction with them at the boundary

## Exit Criteria
- [ ] Every new behavior has a test that was written before the implementation
- [ ] All tests pass (`npm test`, `pytest`, `go test`, or project equivalent)
- [ ] No skipped or disabled tests exist without a linked tracking issue
- [ ] Test names describe the requirement, not the implementation (`rejects_expired_token` not `test_validate_method`)
- [ ] Test pyramid proportions are reasonable (~75% unit, ~20% integration, ~5% E2E)
- [ ] Mocks exist only at system boundaries (network, disk, clock, third-party services)
- [ ] No test depends on another test's execution or shared mutable state
- [ ] Flaky tests are resolved or quarantined with a tracking issue
- [ ] Refactoring steps were performed under green tests only

## Anti-Patterns

| Anti-Pattern | Why It Fails | What to Do Instead |
|---|---|---|
| Writing tests after implementation | Tests confirm what you wrote, not what was required. Bugs in logic are mirrored in tests | Write the test first. Let the failure message guide the implementation |
| Snapshot test overuse | Snapshots assert on **everything**, so they break on irrelevant changes. Developers learn to blindly update them | Assert on specific properties that reflect requirements. Use snapshots only for large stable structures (API schemas, AST outputs) |
| Testing implementation details | Asserting internal method calls or private state couples tests to structure. Any refactor breaks tests without behavior changing | Assert on public outputs, return values, and observable side effects only |
| Shared mutable test state | Test A sets up data that Test B depends on. Reordering or parallelizing tests causes mysterious failures | Each test creates its own fixtures in Arrange and cleans up in teardown |
| Flaky tests tolerated | Teams learn to ignore failures. Real bugs hide in the noise of intermittent test failures | Fix flaky tests immediately. If the fix is non-trivial, quarantine the test and file a tracking issue |
| God test file | One test file with 2,000 lines testing every method of every class | One test file per module or class. Group tests by behavior, not by method name |
| Testing trivial code | Tests for getters, setters, or pass-through functions add maintenance cost with zero defect-detection value | Test behavior that can meaningfully break. A getter that returns a field is not that |
| Mocking everything | When every dependency is mocked, the test runs in a fantasy world. Integration bugs pass unit tests and fail in production | Mock at boundaries. Use real implementations for own code. Prefer fakes over mocks |
| Copy-paste test duplication | Identical test bodies with one parameter changed. When the contract changes, N places need updating | Use parameterized tests or table-driven tests for input/output variations |
| Asserting on error messages | Tests break when someone improves a log message. The message is not the behavior | Assert on error types, status codes, or error categories. Use constants for messages if you must check them |
