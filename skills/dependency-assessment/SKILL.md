---
name: dependency-assessment
capability: Evaluate whether to add, update, or remove a dependency based on maintenance, security, size, and licensing criteria.
---

# Dependency Assessment

## What This Skill Enables

Agents with this skill can make disciplined decisions about third-party packages instead of reflexively running `npm install` or `pip install` at the first sign of complexity. Without it, agents accumulate dependencies that bloat bundles, introduce supply-chain risk, and create upgrade nightmares. A skilled agent treats every new dependency as a long-term commitment and evaluates it accordingly.

## Core Competencies

### 1. Apply the "Do We Really Need This?" Checklist

Before adding any package, answer every question honestly:

| Question | Guidance |
|---|---|
| Does the standard library already cover this? | Check built-ins first. `fetch` exists natively; you may not need `axios`. |
| Is the functionality fewer than ~50 lines to implement? | A `leftPad` situation. Write it yourself. |
| Will this dependency be used in more than one place? | A single call site rarely justifies a new dependency. |
| Does the project already have a similar package? | Two date libraries in one project is a code smell. |
| Is this a direct dependency or only needed at build time? | Dev dependencies carry less runtime risk but still need auditing. |

If you answer "no need" to any of the first four questions, stop. Write the code inline or use what already exists.

### 2. Evaluate Package Health

Never install a package without checking its vital signs:

| Signal | Healthy | Warning | Critical |
|---|---|---|---|
| **Last commit** | Within 3 months | 3-12 months ago | Over 1 year ago |
| **Open issues** | Triaged, responsive maintainers | Growing backlog | Hundreds unaddressed |
| **Weekly downloads** | Stable or growing | Declining steadily | Under 1,000 |
| **Bus factor** | 3+ active contributors | 2 contributors | Single maintainer |
| **Release cadence** | Regular semver releases | Sporadic | No releases in 12+ months |
| **Test suite** | CI passing, good coverage | CI present but flaky | No CI at all |

Use `npm info <pkg>`, GitHub insights, or equivalent tooling to gather these signals. Do not guess.

### 3. Measure Bundle Size Impact

For frontend projects, size is a first-class concern:

```
# Check before installing
npx bundlephobia <package-name>

# After installing, compare
npx webpack-bundle-analyzer stats.json
```

Rule of thumb: if a utility package adds more than 10 KB minified+gzipped for a function you could write in 30 lines, write it yourself. For backend projects, size matters less but startup time and memory footprint still count.

### 4. Verify License Compatibility

Every dependency carries a license. Common compatibility:

| Project License | Compatible Dependency Licenses | Incompatible |
|---|---|---|
| MIT | MIT, BSD, ISC, Apache-2.0 | GPL, AGPL (viral) |
| Apache-2.0 | MIT, BSD, ISC, Apache-2.0 | GPL, AGPL |
| GPL-3.0 | MIT, BSD, ISC, Apache-2.0, GPL | AGPL (depending on use) |
| Proprietary | MIT, BSD, ISC, Apache-2.0 | GPL, AGPL, SSPL |

Run `npx license-checker` or `pip-licenses` and flag anything viral or unknown. Packages with no license declared are legally risky -- treat them as proprietary.

### 5. Decide: Install, Vendor, or Write Your Own

| Strategy | When to Use |
|---|---|
| **Install normally** | Package is healthy, well-maintained, widely used, license-compatible, and provides substantial functionality. |
| **Vendor (copy source into repo)** | Package is small, stable, and unlikely to change, but you want to avoid a network dependency or the package is archived. Pin the version and note the origin. |
| **Write your own** | The needed functionality is small, the package is unhealthy, or you need tight control over behavior and performance. |

## Behavioral Rules

1. Never add a dependency without explicitly evaluating the checklist in competency 1.
2. Always verify the license is compatible with the project before installing.
3. Always check for an existing package in the project that serves the same purpose before adding a new one.
4. Run a security audit (`npm audit`, `pip audit`, `cargo audit`) after adding or updating any dependency.
5. Pin dependency versions in lockfiles and commit the lockfile with the same PR that adds the dependency.

## Failure Modes

| Failure | Symptom | Correction |
|---|---|---|
| Dependency hoarding | `package.json` grows by one package per task. | Apply the "do we need this?" checklist every time. |
| Abandoned dependency | Build breaks months later because an unmaintained package conflicts. | Check health signals before adopting; set calendar reminders to re-evaluate. |
| License violation | Legal flags a GPL dependency in a proprietary product. | Run license checks in CI; block incompatible licenses automatically. |
| Bundle bloat | Page load time degrades after adding a "small" utility. | Measure size impact before merging; set bundle budgets in CI. |
| Duplicate functionality | Two packages that do the same thing coexist in the project. | Audit existing dependencies before proposing a new one; consolidate. |
