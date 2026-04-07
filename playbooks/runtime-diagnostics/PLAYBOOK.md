---
name: runtime-diagnostics
trigger: when verifying rendered output, debugging browser behavior, or profiling runtime performance
---

# Runtime Diagnostics

Browser and runtime verification through live instrumentation, ensuring that what
the user actually sees and experiences matches what the specification demands.

## Objective

Confirm that the running application behaves correctly by inspecting the DOM,
monitoring network traffic, watching for console errors, and profiling
performance -- all before declaring a task complete.

## Triggers

- A UI change has been implemented and needs visual/functional verification.
- A network-dependent feature is added or modified.
- Console errors or warnings appear during manual or automated testing.
- Performance regressions are suspected (slow loads, jank, memory growth).
- An agent needs to verify rendered output via MCP-based browser inspection.

## Workflow

### 1. DOM and Rendered-Output Inspection

- Open browser DevTools (Elements panel) or use an MCP snapshot tool to capture
  the current accessibility tree or DOM structure.
- Compare the rendered output against the specification: correct elements, text
  content, attributes, ARIA roles, and visual ordering.
- Check that conditional UI states (loading, empty, error) render properly by
  driving the application into each state.

### 2. Console Error Monitoring

- Open the Console panel (or retrieve console messages via MCP) **before**
  exercising the feature under test.
- Filter for `error` and `warning` levels.
- Treat any new console error introduced by the change as a defect until proven
  otherwise -- suppressions must be explicitly justified.
- Watch for deprecation warnings that signal upcoming breakage.

### 3. Network Request Validation

- Open the Network tab (or capture requests via MCP network inspection).
- Exercise the feature and verify:
  - **Status codes** -- expected 2xx for success paths; proper 4xx/5xx handling.
  - **Payload shapes** -- request bodies match the API contract; responses
    contain required fields.
  - **Timing** -- no requests hanging beyond acceptable thresholds (e.g., 3 s).
  - **Redundant calls** -- no duplicate fetches caused by re-render loops.
- For agents: use `browser_network_requests` with a URL filter to isolate
  relevant traffic.

### 4. Performance Profiling

- **Long tasks**: Record a Performance trace and flag any task exceeding 50 ms
  on the main thread.
- **Layout shifts**: Check the Cumulative Layout Shift (CLS) score; investigate
  elements that move after initial paint.
- **Memory leaks**: Take heap snapshots before and after repeated navigation
  cycles; compare retained object counts.
- **Resource size**: Confirm that no single JS bundle or image exceeds
  agreed-upon budgets.

### 5. MCP-Based Runtime Inspection for Agents

- Use `browser_snapshot` to obtain a structured accessibility tree instead of
  relying on screenshots alone.
- Use `browser_evaluate` to run assertions directly in the page context
  (e.g., `document.querySelectorAll('.error').length === 0`).
- Use `browser_console_messages` to programmatically check for errors after
  each interaction step.

## Guardrails

| Rule | Detail |
|------|--------|
| Never skip console check | Every verification session must include a console-error scan. |
| Compare against spec, not intuition | Use the written specification as the source of truth. |
| Profile on representative data | Synthetic empty states hide real-world performance issues. |
| Automate where repeatable | If a verification will run more than twice, script it. |
| Do not ship with known console errors | Suppress only with a documented reason and a tracking issue. |

## Exit Criteria

- Rendered output matches the specification for all defined states.
- Network requests return expected status codes and payloads within time budgets.
- Console is free of new errors and warnings at `error` level.
- No long tasks above 50 ms are introduced by the change.
- No layout shifts attributable to the change.
- Memory profile shows no growth across repeated interaction cycles.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Approach |
|--------------|-------------|-----------------|
| Screenshot-only verification | Misses accessibility, hidden elements, and timing issues. | Combine snapshots with DOM/accessibility tree inspection. |
| Ignoring network timing | Slow requests degrade UX silently. | Set explicit latency budgets and alert on violations. |
| Dismissing console warnings | Warnings become errors in future dependency upgrades. | Triage every warning; suppress only with justification. |
| Profiling in dev mode only | Dev builds include extra overhead that masks real profiles. | Always confirm with a production build. |
| One-shot verification | Flaky issues surface only under repetition. | Run verification loops for timing-sensitive features. |
