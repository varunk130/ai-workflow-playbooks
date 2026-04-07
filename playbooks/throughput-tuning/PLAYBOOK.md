---
name: throughput-tuning
trigger: when a task involves performance optimization, slow page loads, high latency endpoints, large bundle sizes, expensive database queries, or caching strategy decisions
---

# Throughput Tuning Playbook

## Objective

Deliver measurable performance improvements by profiling first and optimizing second. This playbook enforces a measure-first discipline: the agent must never apply an optimization without a measurement that identifies the specific bottleneck being addressed.

## The Rule

**Never optimize without a measurement showing the bottleneck.** Intuition about performance is frequently wrong. Every change in this playbook must be preceded by a profiling result, benchmark number, or monitoring metric that justifies it. If you cannot point to a number, you are not ready to optimize.

## Triggers

Activate this playbook when any of the following appear in the task:

- A performance issue is reported with measurable symptoms (slow load, high TTFB, timeout).
- Bundle size exceeds the project's performance budget.
- A database query appears in slow-query logs or exceeds 100ms.
- Core Web Vitals scores fall below acceptable thresholds.
- A code review flags an N+1 query, missing index, or unoptimized asset.
- The task explicitly requests performance tuning or load testing.

## Core Web Vitals Targets

Use these as the baseline goals for frontend work:

| Metric | Target | What It Measures |
|---|---|---|
| **LCP** (Largest Contentful Paint) | <= 2.5 seconds | How fast the main content becomes visible |
| **INP** (Interaction to Next Paint) | <= 200 milliseconds | How responsive the page is to user interaction |
| **CLS** (Cumulative Layout Shift) | <= 0.1 | How much the layout shifts unexpectedly during load |

If the project defines stricter budgets, use those instead.

## Workflow

### Step 1 -- Establish a Baseline

Before changing anything, measure the current state:

- **Frontend:** Run Lighthouse or WebPageTest. Record LCP, INP, CLS, total bundle size, and time-to-interactive.
- **Backend:** Capture p50, p95, and p99 latency for the relevant endpoints. Record query execution times from slow-query logs or APM tools.
- **Document the numbers.** Paste them into the PR description or task comments. These are the "before" values that justify every subsequent change.

### Step 2 -- Identify the Bottleneck

Analyze the baseline to locate the single largest contributor to the problem:

- **Frontend profiling:** Use Chrome DevTools Performance panel to find long tasks. Use the Coverage tab to identify unused JavaScript and CSS.
- **Backend profiling:** Use language-native profilers (cProfile, pprof, dotTrace) or APM flame graphs to find hot paths.
- **Database profiling:** Run `EXPLAIN ANALYZE` on slow queries. Look for sequential scans on large tables, missing indexes, and excessive row counts.

Do not proceed to optimization until you can state: "The bottleneck is X, and it accounts for Y ms / Y KB of the problem."

### Step 3 -- Optimize the Frontend (if applicable)

Apply only the techniques that address the measured bottleneck:

- **Bundle analysis:** Run `webpack-bundle-analyzer` or equivalent. Identify the largest chunks and look for duplicated or unnecessary dependencies.
- **Code splitting and lazy loading:** Split routes and heavy components so they load on demand. Use dynamic `import()` for anything not needed on initial render.
- **Image optimization:** Serve images in WebP/AVIF format. Use `srcset` and `sizes` for responsive delivery. Lazy-load below-the-fold images with `loading="lazy"`.
- **Render performance:** Eliminate layout thrashing by batching DOM reads and writes. Avoid forced synchronous layouts. Use `will-change` sparingly and only on elements that actually animate.
- **Font loading:** Use `font-display: swap`. Preload critical fonts. Subset fonts to include only required character ranges.

### Step 4 -- Optimize the Backend (if applicable)

Apply only the techniques that address the measured bottleneck:

- **Query optimization:** Add indexes for columns in WHERE, JOIN, and ORDER BY clauses. Rewrite N+1 patterns into batch queries or JOINs. Use `SELECT` only for needed columns.
- **Caching strategies:** Apply caching at the right layer:
  - **Application cache** (in-memory, Redis): for computed results, session data, or API responses that tolerate staleness.
  - **HTTP cache** (Cache-Control, ETag): for static assets and API responses with predictable freshness.
  - **Database query cache:** use cautiously; often better to optimize the query itself.
- **Connection pooling:** Configure pool size based on expected concurrency. Reuse database and HTTP connections rather than creating new ones per request.
- **Pagination:** Never return unbounded result sets. Implement cursor-based or offset pagination with a maximum page size. Default to small pages (20-50 items).
- **Async processing:** Move long-running work (email, image processing, report generation) to background job queues. Return 202 Accepted and let the client poll or subscribe.

### Step 5 -- Enforce Performance Budgets

Set concrete limits and fail the build when they are exceeded:

- **Bundle size budget:** Define maximum sizes for JavaScript and CSS bundles in CI. Tools: `bundlesize`, `size-limit`, Lighthouse CI budgets.
- **Endpoint latency budget:** Alert when p95 latency exceeds the defined threshold in staging or production.
- **Build-time checks:** Add a CI step that runs Lighthouse and fails if LCP > 2.5s or total JS > the defined cap.

Example CI snippet:
```yaml
- name: Check bundle size
  run: npx size-limit
  # Fails the build if any entry exceeds the limit defined in .size-limit.json
```

### Step 6 -- Measure the Result

After applying the optimization, re-run the exact same measurements from Step 1:

- Record the new numbers alongside the old ones.
- Calculate the percentage improvement for each metric.
- If the improvement is less than 10% or does not move the metric past the target threshold, reconsider whether the change is worth the added complexity.
- Include before/after numbers in the PR description.

### Step 7 -- Document and Guard

- Add a brief performance note to the PR explaining what was measured, what was changed, and the resulting improvement.
- Ensure CI budgets are updated to lock in the gains (prevent regression).
- If a new caching layer was added, document the invalidation strategy and TTL rationale.

## Guardrails

- Never skip the baseline measurement. "It felt slow" is not a measurement.
- Never cache without defining an invalidation strategy. Stale data is a bug.
- Never add an index without checking write-path impact on insert-heavy tables.
- Never inline large assets to "save a request." This defeats caching entirely.
- Never set `Cache-Control: no-store` as a blanket policy. Be specific about what should and should not be cached.

## Exit Criteria

- [ ] Baseline measurements are recorded and included in the PR.
- [ ] Each optimization targets a specific, measured bottleneck.
- [ ] After-measurements show improvement that meets or moves toward the target.
- [ ] Performance budgets are configured in CI and passing.
- [ ] No unbounded queries or unpaginated endpoints remain.
- [ ] Caching layers have documented invalidation strategies and TTLs.
- [ ] Before/after numbers are included in the PR description.

## Anti-Patterns

| Anti-Pattern | Why It Fails | Correct Approach |
|---|---|---|
| Optimizing without profiling | You fix the wrong thing and add complexity for no gain | Profile first; optimize the measured bottleneck only |
| Caching everything with long TTLs | Users see stale data; invalidation bugs multiply | Cache selectively with short TTLs; define invalidation rules upfront |
| Adding indexes on every column | Write performance degrades; storage bloats | Index only columns that appear in slow-query WHERE/JOIN/ORDER BY |
| Premature code splitting | Too many small chunks increase request overhead | Split at route boundaries and for genuinely large dependencies |
| Disabling SSR to "simplify" performance | LCP and SEO suffer; initial load becomes blank screen | Keep SSR for critical content; hydrate interactivity progressively |
| Returning full objects from every API | Payload size grows; serialization cost increases | Return only requested fields; use sparse fieldsets or GraphQL projections |
| Setting infinite `max-age` without versioned URLs | Clients cache stale assets with no way to bust the cache | Use content-hashed filenames with long `max-age`; purge on deploy |
