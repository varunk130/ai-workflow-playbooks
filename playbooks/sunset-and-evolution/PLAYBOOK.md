---
name: sunset-and-evolution
trigger: When deprecating a feature, removing dead code, or migrating consumers to a replacement
---

# Sunset and Evolution

Managed deprecation and migration so the codebase stays lean and maintainable.

## Objective

Retire outdated code, APIs, and features with minimal disruption to consumers.
Treat code as a liability: every line that no longer earns its keep increases
maintenance cost, cognitive load, and security surface area.

## Triggers

- A feature or API has a newer replacement ready for adoption.
- Usage metrics show a component is unused or near-zero traffic.
- A dependency reaches end-of-life or introduces a security concern.
- Technical debt review identifies dead or duplicated code.
- A strategic architecture decision makes an existing module obsolete.

## Workflow

### 1. Classify the Deprecation

| Type | Definition | Expectation |
|---|---|---|
| **Compulsory** | Hard deadline; the old path will be removed on a fixed date. | Consumers must migrate before the cutoff. |
| **Advisory** | Soft guidance; the old path still works but is no longer recommended. | Consumers should plan migration at their convenience. |

Choose compulsory when security, cost, or architectural integrity demands it.
Default to advisory when the old path is stable and low-risk.

### 2. Communicate Early and Often

- Publish a deprecation notice: **what** is changing, **why**, **when**, and the
  **recommended alternative**.
- Notify consumers through every relevant channel: changelog, in-code deprecation
  annotations, API response headers (`Deprecation`, `Sunset`), email, and Slack.
- Provide a migration guide with before/after examples.
- Set a timeline: minimum **one full release cycle** notice for compulsory changes.

### 3. Choose a Migration Pattern

| Pattern | When to Use |
|---|---|
| **Strangler Fig** | Gradually route traffic from old to new; old shrinks over time. |
| **Parallel Run** | Run old and new side-by-side, compare outputs, switch when confident. |
| **Feature Flag Cutover** | Toggle between old and new at runtime; instant rollback capability. |

### 4. Execute the Migration

- Instrument the old path with usage telemetry to track remaining consumers.
- Provide automated codemods or migration scripts where feasible.
- Offer office hours or pairing sessions for consumers who need help.
- Monitor error rates and support tickets during the transition window.

### 5. Remove Dead Code

- Once telemetry confirms zero usage (or the hard deadline passes), delete the
  old implementation entirely.
- Remove associated tests, configuration, documentation references, and feature flags.
- Treat removal as a first-class change: a dedicated PR with clear rationale.
- Run a dependency analysis to ensure no hidden references remain.
- Update import maps, dependency injection registrations, and route tables.
- Celebrate removal: fewer lines of code means less surface area to maintain,
  test, and secure.

## Guardrails

- **Never remove without notice.** Even internal code deserves a deprecation period.
- **Deprecation annotations are mandatory** (`@deprecated`, `Obsolete`, or framework
  equivalent) with a pointer to the replacement.
- **Telemetry must validate zero usage** before final removal in advisory deprecations.
- **Rollback plan exists** for every migration step.
- **One migration pattern per deprecation** -- do not mix strategies mid-flight.

## Exit Criteria

- [ ] Deprecation notice published with timeline and alternative.
- [ ] All known consumers have migrated or acknowledged the timeline.
- [ ] Usage telemetry confirms zero (or acceptable residual) traffic on the old path.
- [ ] Dead code, tests, config, and docs for the old path are fully removed.
- [ ] No orphaned feature flags remain.
- [ ] Post-removal monitoring shows no regressions.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Alternative |
|---|---|---|
| Silent removal without notice | Breaks consumers, erodes trust | Publish deprecation with timeline and guide |
| "Deprecated" label with no removal date | Code lingers forever, false sense of action | Set a concrete sunset date, even if generous |
| Keeping dead code "just in case" | Maintenance burden, misleading grep results | Delete it; version control remembers |
| Big-bang migration on a single day | High risk, stressful, no rollback margin | Incremental migration with strangler fig or flags |
| Skipping telemetry before removal | Unknown consumers break silently | Instrument usage and verify zero traffic first |
