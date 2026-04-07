Run the CONSTRUCT stage of the AI Workflow Playbooks pipeline.

Load and execute these playbooks together:
1. `playbooks/iterative-construction/PLAYBOOK.md` — Build in thin vertical slices with continuous verification
2. `playbooks/test-first-engineering/PLAYBOOK.md` — Write tests before implementation code

For each task in the implementation sequence:
- Write a failing test that captures the expected behavior
- Implement the minimum code to make the test pass
- Refactor for clarity while keeping tests green
- Commit after each completed task

Verify all tests pass and no regressions are introduced before marking a task complete.
