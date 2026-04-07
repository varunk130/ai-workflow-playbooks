---
name: architecture-and-decomposition
trigger: Use when breaking a specification into implementable tasks. Activate after spec authoring, before construction begins.
---

# Architecture & Decomposition

## Objective
Transform a specification into an ordered sequence of small, independently verifiable work units. Good decomposition means each task can be implemented, tested, and reviewed without requiring the entire system to exist.

## Triggers
- A specification is complete and approved
- A large feature needs to be broken into manageable pieces
- Work needs to be parallelized across multiple agents
- Dependencies between components need to be mapped

## Workflow

### Step 1: Identify System Boundaries
From the spec, extract:
- Major components or modules involved
- External systems and integration points
- Data flows between components
- Shared state or resources

Sketch the component relationships (use text diagrams):
```
[Component A] --depends on--> [Component B]
[Component B] --calls--> [External API]
[Component C] --reads--> [Shared Database]
```

### Step 2: Define the Task Inventory
For each functional requirement in the spec, create a task:

```
TASK: [Short descriptive title]
REQUIREMENT: [Which spec requirement this implements]
INPUT: [What must exist before this task starts]
OUTPUT: [What this task produces — file, endpoint, test, etc.]
ACCEPTANCE: [How to verify this task is complete]
SIZE: [S/M/L — aim for S or M]
```

Rules for task sizing:
- **Small (S)**: Implementable in a single focused session, ~50-100 lines changed
- **Medium (M)**: May touch 2-3 files, ~100-300 lines changed
- **Large (L)**: Must be decomposed further — L tasks are plans, not tasks

### Step 3: Establish Dependency Order
Arrange tasks in implementation order:
1. Foundation tasks (data models, configuration, shared utilities)
2. Core logic tasks (business rules, algorithms, state management)
3. Integration tasks (API endpoints, UI components, external calls)
4. Polish tasks (error handling, edge cases, performance)

For each task, explicitly state:
- **Blocked by**: Which tasks must complete first
- **Enables**: Which tasks can start after this one

### Step 4: Identify the Critical Path
Mark the longest chain of dependent tasks. This is your critical path — delays here delay everything. Consider:
- Can any critical-path tasks be simplified to reduce risk?
- Are there parallel tracks that can proceed independently?
- Where are the highest-risk tasks? Schedule them early

### Step 5: Create the Implementation Sequence
Output the final ordered task list:

```
## Implementation Sequence

### Phase 1: Foundation
- [ ] Task 1.1: [title] (blocked by: none)
- [ ] Task 1.2: [title] (blocked by: none)

### Phase 2: Core
- [ ] Task 2.1: [title] (blocked by: 1.1)
- [ ] Task 2.2: [title] (blocked by: 1.1, 1.2)

### Phase 3: Integration
- [ ] Task 3.1: [title] (blocked by: 2.1)

### Phase 4: Polish
- [ ] Task 4.1: [title] (blocked by: 3.1)
```

## Guardrails
- Every task must be independently testable. If a task can only be verified by running the full system, decompose further
- No task should modify more than 5 files. Larger changes indicate insufficient decomposition
- Dependencies must be explicit — no hidden coupling between tasks
- If a task description starts with "and also," it's two tasks

## Exit Criteria
- [ ] Every spec requirement maps to at least one task
- [ ] No task is sized Large — all are Small or Medium
- [ ] Dependencies are explicitly documented
- [ ] Critical path is identified
- [ ] Each task has clear acceptance criteria
- [ ] The implementation sequence is ordered and ready for construction

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| "We'll figure out the order as we go" | Agents waste cycles discovering implicit dependencies |
| One giant task per component | No incremental verification; bugs hide until integration |
| Skipping acceptance criteria | No way to know when a task is actually done |
| Parallel tasks with hidden shared state | Race conditions and integration failures |
| Leaving the hardest task for last | Risk compounds; discover blockers early |
