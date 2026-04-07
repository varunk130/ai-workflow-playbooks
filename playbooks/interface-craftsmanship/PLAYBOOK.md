---
name: interface-craftsmanship
trigger: Use when building frontend components, UI layouts, or user-facing interfaces. Activate during construction of any visual layer.
---

# Interface Craftsmanship

## Objective
Build frontend interfaces that are accessible, maintainable, and performant by default. Good interfaces emerge from systematic component design, not ad-hoc styling.

## Triggers
- Building new UI components or pages
- Refactoring an existing frontend codebase
- Integrating a design system or component library
- Addressing accessibility or responsive design requirements

## Do Not Use When
- Building backend-only services or CLI tools
- The task is purely API or data layer work

## Workflow

### Step 1: Audit the Existing System
Before creating new components:
1. Inventory existing components — what already exists?
2. Check for a design system or component library in use
3. Identify patterns: how is state managed? What styling approach is used?
4. Note the accessibility baseline — are ARIA roles present? Is keyboard navigation supported?

### Step 2: Design the Component Architecture
For each new UI element:

```
COMPONENT: [Name]
PURPOSE: [Single responsibility — what it renders and why]
PROPS/INPUTS: [Data it receives]
STATE: [Internal state it manages, if any]
EVENTS: [User interactions it handles]
ACCESSIBILITY: [Keyboard behavior, ARIA roles, screen reader text]
```

Principles:
- **Single responsibility**: One component does one thing
- **Composition over configuration**: Small components composed together beat large configurable ones
- **Props down, events up**: Data flows down; user actions bubble up
- **Stateless by default**: Only hold state when you must

### Step 3: Implement with Accessibility First
Build the semantic structure before styling:

1. **Write semantic HTML** — Use `<button>`, `<nav>`, `<main>`, `<article>` before reaching for `<div>`
2. **Add keyboard support** — Every interactive element must be reachable via Tab and activatable via Enter/Space
3. **Include ARIA attributes** — Labels, roles, live regions where semantic HTML isn't sufficient
4. **Handle focus management** — Modals trap focus; route changes move focus to new content
5. **Test without a mouse** — Navigate the entire flow using only the keyboard

### Step 4: Apply Styling Systematically
Follow the project's established pattern (CSS modules, Tailwind, styled-components, etc.):
- Use design tokens for colors, spacing, typography — never hardcode values
- Responsive layouts use relative units and container queries or media queries
- Dark mode and theme support through CSS custom properties or theme context
- Animations respect `prefers-reduced-motion`

### Step 5: Manage State Deliberately
State management decisions by scope:

| Scope | Strategy |
|-------|----------|
| Component-local | `useState` / internal reactive state |
| Shared between siblings | Lift to nearest common parent |
| Feature-wide | Context provider or feature store |
| Application-wide | Global store (Redux, Zustand, Pinia, etc.) |
| Server-derived | Server state library (React Query, SWR, etc.) |

Rules:
- Derive computed values — don't store what you can calculate
- URL state is state too — use query params for shareable UI state
- Form state belongs to the form library, not your global store

### Step 6: Write Component Tests
For each component:
1. Render with default props → verify it mounts without error
2. Render with edge-case props → empty data, long strings, missing optional props
3. Simulate user interactions → click, type, submit
4. Assert on visible output → text content, ARIA attributes, DOM structure
5. Do NOT assert on implementation details → class names, internal state, snapshot matching

### Step 7: Verify Across Contexts
- Test at multiple viewport widths (320px, 768px, 1024px, 1440px)
- Verify in both light and dark themes (if applicable)
- Run an accessibility audit (axe-core, Lighthouse accessibility score)
- Check loading states, error states, and empty states

## Guardrails
- Never use `div` with an `onClick` handler — use `button` or `a` with proper roles
- Do not disable the browser's default focus indicators without providing custom ones
- Images must have `alt` text. Decorative images use `alt=""`
- Color must not be the only means of conveying information
- Do not suppress TypeScript errors in component props — fix the type contract

## Exit Criteria
- [ ] Components follow single-responsibility principle
- [ ] All interactive elements are keyboard accessible
- [ ] ARIA attributes are present where semantic HTML is insufficient
- [ ] Design tokens are used (no hardcoded colors or spacing)
- [ ] Component tests cover rendering, interaction, and edge cases
- [ ] Accessibility audit passes with zero critical violations
- [ ] Responsive behavior verified at standard breakpoints

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| "We'll add accessibility later" | Retrofitting a11y is 5-10x more expensive than building it in |
| Hardcoding pixel values for spacing | Breaks when themes, zoom levels, or viewports change |
| One massive component with conditional rendering | Untestable, unreadable, unmaintainable |
| Testing implementation details (class names, state) | Tests break on refactoring without behavior change |
| Copying component code instead of composing | Duplication diverges silently over time |
| "It works on my screen" | Does not account for zoom, screen readers, or small viewports |
