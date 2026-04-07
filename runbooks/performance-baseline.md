# Performance Baseline Runbook

Reference checklist for performance optimization. Load this when tuning throughput or diagnosing performance issues.

## Core Web Vitals Targets

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP (Largest Contentful Paint) | ≤2.5s | 2.5s–4.0s | >4.0s |
| INP (Interaction to Next Paint) | ≤200ms | 200ms–500ms | >500ms |
| CLS (Cumulative Layout Shift) | ≤0.1 | 0.1–0.25 | >0.25 |

## Frontend Performance Checklist

### Bundle Size
- [ ] JavaScript bundle analyzed (webpack-bundle-analyzer, source-map-explorer)
- [ ] Code splitting applied for route-level chunks
- [ ] Tree shaking verified — unused exports are eliminated
- [ ] No duplicate dependencies in the bundle
- [ ] Bundle size budget enforced in CI (<250KB gzipped for main bundle)

### Loading Performance
- [ ] Critical CSS inlined or loaded with high priority
- [ ] Non-critical CSS and JS loaded asynchronously
- [ ] Images use modern formats (WebP, AVIF) with fallbacks
- [ ] Images are lazy-loaded below the fold
- [ ] Images include explicit `width` and `height` (prevents CLS)
- [ ] Fonts use `font-display: swap` and are preloaded

### Runtime Performance
- [ ] No layout thrashing (interleaved reads and writes to the DOM)
- [ ] Long lists use virtualization (react-window, vue-virtual-scroller)
- [ ] Expensive computations are memoized or debounced
- [ ] Animations use `transform` and `opacity` (GPU-accelerated properties)
- [ ] `requestAnimationFrame` used for visual updates, not `setInterval`
- [ ] Event handlers on scroll/resize are throttled

### Rendering
- [ ] Unnecessary re-renders identified and eliminated (React Profiler, Vue devtools)
- [ ] Component memoization applied where measurement shows benefit
- [ ] Context providers scoped narrowly to minimize consumer re-renders
- [ ] Server-side rendering or static generation used where appropriate

## Backend Performance Checklist

### Database
- [ ] Queries analyzed with EXPLAIN — no full table scans on large tables
- [ ] Indexes exist for all WHERE, JOIN, and ORDER BY columns
- [ ] N+1 query patterns eliminated (use eager loading or batching)
- [ ] Connection pooling configured with appropriate min/max limits
- [ ] Slow query logging enabled (>100ms threshold)

### Caching
- [ ] Cache strategy defined: read-through, write-through, or cache-aside
- [ ] Cache TTLs set based on data freshness requirements
- [ ] Cache invalidation is explicit, not time-based only
- [ ] Cache stampede protection in place (locking or request coalescing)
- [ ] HTTP caching headers set for static assets (`Cache-Control`, `ETag`)

### API Performance
- [ ] Pagination implemented for all list endpoints
- [ ] Response payload minimized — only return requested fields
- [ ] Compression enabled (gzip/brotli) for API responses
- [ ] Rate limiting configured to protect against abuse
- [ ] Timeouts set on all downstream calls

### Infrastructure
- [ ] Horizontal scaling tested and verified
- [ ] Auto-scaling rules configured with appropriate thresholds
- [ ] CDN configured for static assets and cacheable API responses
- [ ] Database read replicas used for read-heavy workloads

## Measurement Commands

```bash
# Lighthouse audit
npx lighthouse https://example.com --output=json --output-path=./report.json

# Bundle analysis (webpack)
npx webpack-bundle-analyzer stats.json

# Bundle analysis (vite)
npx vite-bundle-visualizer

# Database slow queries (PostgreSQL)
SELECT query, calls, mean_exec_time FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 20;

# Node.js profiling
node --prof app.js
node --prof-process isolate-*.log > profile.txt

# Load testing
npx autocannon -c 100 -d 30 http://localhost:3000/api/users
```

## Performance Budget Template

```json
{
  "budgets": [
    {
      "resourceType": "script",
      "budget": 250000
    },
    {
      "resourceType": "stylesheet",
      "budget": 50000
    },
    {
      "resourceType": "image",
      "budget": 500000
    },
    {
      "metric": "largest-contentful-paint",
      "budget": 2500
    },
    {
      "metric": "interaction-to-next-paint",
      "budget": 200
    }
  ]
}
```
