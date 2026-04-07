---
name: version-control-discipline
trigger: When starting any feature branch, preparing a commit, or opening a pull request
---

# Version Control Discipline

Atomic commits, trunk-based flow, and clean history for sustainable collaboration.

## Objective

Maintain a repository history that tells a coherent story of the project's evolution.
Every commit should be a self-contained, deployable unit that a future reader can
understand without additional context. Trunk stays releasable at all times.

## Triggers

- A new feature, bugfix, or refactor is about to begin.
- A developer is preparing to commit or push changes.
- A pull request is being authored or reviewed.
- The main branch has diverged significantly from a long-lived branch.

## Workflow

### 1. Branch from Trunk

- Create a short-lived branch off `main` (or the designated trunk).
- Branch naming convention: `<type>/<ticket>-<slug>`.
  - Examples: `feat/APP-142-user-avatar`, `fix/APP-305-null-cart`, `chore/APP-410-upgrade-deps`.
  - Types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`.
- Target lifetime: **< 2 days**. If the work is larger, decompose it into stacked PRs
  or use feature flags to merge incomplete but safe code.

### 2. Make Atomic Commits

- Each commit represents **one logical change** -- a single reason to exist.
- A commit must leave the codebase in a compilable, test-passing state.
- Group related file changes together: if a function rename touches three files,
  that is one commit, not three.
- Never bundle unrelated changes (e.g., a bugfix and a dependency upgrade).
- During iterative construction, use **commit-as-checkpoint**: commit frequently with
  a `wip:` prefix, then interactive-rebase to squash and rewrite before opening the PR.
- The checkpoint pattern gives you undo points while working, and the squash gives
  reviewers a clean, intentional history.

### 3. Write Meaningful Commit Messages

```
<type>(<scope>): <imperative summary>   (50 chars soft limit)

<body>                                   (72 chars per line)
Explain WHAT changed and WHY. Omit HOW -- the diff shows that.

Refs: APP-142
```

- Use imperative mood: "Add filter", not "Added filter" or "Adds filter".
- Reference the ticket ID so traceability is automatic.
- If the commit reverts another, state the reverted SHA in the body.

### 4. Size the Change

- Target **100 -- 300 changed lines** per pull request.
- Larger changes increase review fatigue and defect escape rate.
- If a PR exceeds 400 lines, split it: extract a preparatory refactor PR first.

### 5. Choose the Right Merge Strategy

| Situation | Strategy | Reason |
|---|---|---|
| Single-author feature branch, few commits | **Rebase + fast-forward** | Linear history, easy bisect |
| Multi-author branch or stacked PRs | **Merge commit** | Preserves collaboration context |
| Long-running release branch back-merge | **Merge commit (no fast-forward)** | Clear integration point |

### 6. Open the Pull Request

- Fill in the PR template: summary, motivation, testing notes, screenshots if UI.
- Link the originating ticket.
- Request review from at least one domain-knowledgeable teammate.
- Ensure CI is green before requesting review.

## Guardrails

- **No direct pushes to trunk.** All changes arrive via reviewed pull requests.
- **No force-pushes to shared branches.** Rewrite only your own unpublished history.
- **CI must pass** on the branch before merge is permitted.
- **Secrets never enter version control.** Use `.gitignore`, pre-commit hooks, and
  secret-scanning tooling to enforce this.
- Branch protection rules enforce required reviews and status checks.

## Exit Criteria

- [ ] Branch is short-lived (merged or closed within 2 business days).
- [ ] Every commit is atomic and passes CI independently.
- [ ] Commit messages follow the imperative format and reference a ticket.
- [ ] PR is within the 100-300 line target range.
- [ ] The chosen merge strategy matches the collaboration context.
- [ ] No secrets, generated files, or unrelated changes are included.

## Anti-Patterns

| Anti-Pattern | Why It Hurts | Better Alternative |
|---|---|---|
| "WIP" commits left in final history | Noise in blame/bisect, no meaningful message | Squash WIP commits before merge |
| Mega-PRs (>500 lines) | Rubber-stamp reviews, hidden defects | Decompose into stacked or preparatory PRs |
| Long-lived feature branches (>1 week) | Painful merges, integration risk | Trunk-based flow with feature flags |
| Vague messages ("fix stuff", "updates") | Lost context for future maintainers | Imperative message with what + why |
| Mixing refactor and feature in one commit | Hard to revert or review independently | Separate refactor commit first |
| Force-pushing shared branches | Destroys teammates' local state | Coordinate or use merge commits |
