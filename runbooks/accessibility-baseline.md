# Accessibility Baseline Runbook

Reference checklist for WCAG 2.1 AA compliance. Load this when building or auditing user interfaces for accessibility.

## Keyboard Navigation

- [ ] All interactive elements reachable via Tab key
- [ ] Tab order follows visual layout (logical reading order)
- [ ] Focus indicator is visible on every interactive element
- [ ] Custom focus styles meet 3:1 contrast ratio against adjacent colors
- [ ] Escape key closes modals, dropdowns, and popups
- [ ] Arrow keys navigate within composite widgets (tabs, menus, radio groups)
- [ ] No keyboard traps — user can always Tab away from any element
- [ ] Skip navigation link available for repetitive content

## Focus Management

- [ ] Focus moves to modal content when modal opens
- [ ] Focus returns to trigger element when modal closes
- [ ] Focus is trapped inside modals (Tab does not escape to background content)
- [ ] Route changes move focus to the new page content or heading
- [ ] Dynamic content additions are announced or focused appropriately

## Screen Reader Support

- [ ] All images have descriptive `alt` text (or `alt=""` for decorative images)
- [ ] Form inputs have associated `<label>` elements (via `for`/`id` or nesting)
- [ ] Buttons and links have descriptive accessible names
- [ ] Headings use proper hierarchy (`h1` → `h2` → `h3`, no skipping levels)
- [ ] Landmark regions used (`<main>`, `<nav>`, `<aside>`, `<header>`, `<footer>`)
- [ ] Tables have `<th>` headers with `scope` attributes
- [ ] Dynamic content updates use `aria-live` regions
- [ ] Error messages are associated with their form fields via `aria-describedby`

## ARIA Usage

Use ARIA only when semantic HTML is insufficient:
- [ ] `role` attributes match the element's purpose
- [ ] `aria-label` or `aria-labelledby` provides accessible names where visible text is absent
- [ ] `aria-expanded` indicates expandable widgets (accordions, dropdowns)
- [ ] `aria-selected` indicates selected items in lists and tabs
- [ ] `aria-hidden="true"` used on decorative or duplicative content
- [ ] `aria-describedby` links elements to descriptive text
- [ ] `aria-required` marks required form fields (in addition to visual indicators)

## Visual Design

### Color and Contrast
- [ ] Text meets 4.5:1 contrast ratio against background (normal text)
- [ ] Large text (18px+ or 14px+ bold) meets 3:1 contrast ratio
- [ ] UI components and graphics meet 3:1 contrast ratio
- [ ] Color is never the sole means of conveying information
- [ ] Links are distinguishable from surrounding text (not just by color)

### Typography and Layout
- [ ] Text can be resized to 200% without loss of content
- [ ] Content reflows at 320px viewport width (no horizontal scrolling)
- [ ] Line height is at least 1.5x font size for body text
- [ ] Paragraph spacing is at least 2x font size
- [ ] No text is embedded in images (except logos)

### Motion and Animation
- [ ] Animations respect `prefers-reduced-motion` media query
- [ ] No content flashes more than 3 times per second
- [ ] Auto-playing media can be paused or stopped
- [ ] Carousels have pause controls and visible navigation

## Forms

- [ ] Every input has a visible label
- [ ] Required fields are indicated visually AND programmatically
- [ ] Error messages are specific ("Email format invalid" not "Error")
- [ ] Error messages appear near the field they describe
- [ ] Form submission errors provide a summary at the top of the form
- [ ] Autocomplete attributes used for common fields (name, email, address)

## Testing Tools

```bash
# Automated accessibility audit
npx axe-core-cli https://localhost:3000

# Lighthouse accessibility score
npx lighthouse https://localhost:3000 --only-categories=accessibility

# Color contrast checker
# Use browser DevTools → Elements → Styles → Color contrast ratio
```

### Manual Testing Checklist
1. Navigate the entire page using only the keyboard
2. Use a screen reader (VoiceOver, NVDA, JAWS) to navigate
3. Zoom to 200% and verify content remains usable
4. Enable high contrast mode and verify readability
5. Disable images and verify `alt` text conveys the information
6. Test with `prefers-reduced-motion` enabled
