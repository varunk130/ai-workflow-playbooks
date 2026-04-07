# Testing Patterns Runbook

Reference checklist for common testing patterns. Load this runbook when you need specific testing guidance beyond what the Test-First Engineering playbook provides.

## Test Structure: Arrange-Act-Assert

Every test follows three phases:

```javascript
test('calculates total with tax', () => {
  // Arrange — set up test data and preconditions
  const items = [{ price: 10 }, { price: 20 }];
  const taxRate = 0.08;

  // Act — execute the behavior under test
  const total = calculateTotal(items, taxRate);

  // Assert — verify the expected outcome
  expect(total).toBe(32.40);
});
```

## Naming Conventions

Test names should read as behavior documentation:

```
// Pattern: [unit] [behavior] when [condition]
'cart calculates subtotal when items have varying quantities'
'login rejects credentials when password is expired'
'search returns empty results when query matches nothing'
```

## Common Assertions

### Equality
```javascript
expect(result).toBe(expected);          // strict equality
expect(result).toEqual(expected);       // deep equality for objects
expect(result).toBeCloseTo(0.3, 5);    // floating point
```

### Truthiness
```javascript
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeDefined();
```

### Collections
```javascript
expect(array).toHaveLength(3);
expect(array).toContain(item);
expect(array).toEqual(expect.arrayContaining([a, b]));
expect(object).toHaveProperty('key', 'value');
```

### Errors
```javascript
expect(() => riskyOperation()).toThrow();
expect(() => riskyOperation()).toThrow('specific message');
expect(asyncFn()).rejects.toThrow(CustomError);
```

## Mocking Patterns

### Mock at Boundaries, Not Internals
```javascript
// GOOD — mock the external HTTP call
jest.mock('./api-client');
apiClient.fetchUser.mockResolvedValue({ id: 1, name: 'Test' });

// BAD — mock an internal helper function
jest.mock('./utils/formatName');  // this is implementation detail
```

### Mock Functions
```javascript
const callback = jest.fn();
callback.mockReturnValue(42);
callback.mockImplementation((x) => x * 2);

expect(callback).toHaveBeenCalledWith('arg');
expect(callback).toHaveBeenCalledTimes(1);
```

### Timer Mocks
```javascript
jest.useFakeTimers();
scheduleRetry();
jest.advanceTimersByTime(5000);
expect(retryHandler).toHaveBeenCalled();
jest.useRealTimers();
```

## Component Testing (React / Testing Library)

```javascript
import { render, screen, fireEvent } from '@testing-library/react';

test('submit button is disabled when form is invalid', () => {
  render(<LoginForm />);

  const submitButton = screen.getByRole('button', { name: /submit/i });
  expect(submitButton).toBeDisabled();

  fireEvent.change(screen.getByLabelText(/email/i), {
    target: { value: 'user@example.com' }
  });
  fireEvent.change(screen.getByLabelText(/password/i), {
    target: { value: 'securepassword' }
  });

  expect(submitButton).toBeEnabled();
});
```

Principles:
- Query by role, label, or text — never by test ID unless no semantic alternative exists
- Simulate user interactions (click, type) rather than calling component methods
- Assert on visible output, not internal state

## API / Integration Testing

```javascript
import request from 'supertest';
import app from '../app';

test('POST /users creates a user and returns 201', async () => {
  const response = await request(app)
    .post('/users')
    .send({ name: 'Test User', email: 'test@example.com' })
    .expect(201);

  expect(response.body).toMatchObject({
    name: 'Test User',
    email: 'test@example.com',
  });
  expect(response.body.id).toBeDefined();
});

test('POST /users returns 400 for missing email', async () => {
  const response = await request(app)
    .post('/users')
    .send({ name: 'Test User' })
    .expect(400);

  expect(response.body.error.code).toBe('VALIDATION_FAILED');
});
```

## E2E Testing (Playwright)

```javascript
import { test, expect } from '@playwright/test';

test('user can complete checkout flow', async ({ page }) => {
  await page.goto('/products');
  await page.getByRole('button', { name: 'Add to Cart' }).first().click();
  await page.getByRole('link', { name: 'Cart' }).click();
  await page.getByRole('button', { name: 'Checkout' }).click();

  await expect(page.getByText('Order confirmed')).toBeVisible();
});
```

## Test Anti-Patterns

| Anti-Pattern | Problem | Solution |
|-------------|---------|----------|
| Snapshot everything | Tests pass but don't verify behavior; break on any change | Assert on specific values and behaviors |
| Shared mutable state | Tests pass individually, fail together | Fresh setup in each test (beforeEach) |
| Testing implementation | Tests break on refactoring | Test inputs and outputs, not internals |
| No assertion | Test passes but verifies nothing | Every test must have at least one meaningful assertion |
| Sleep-based waits | Flaky timing; slow test suite | Use waitFor, polling, or event-based synchronization |
| God test | One test verifies everything | One concept per test; multiple assertions only for related checks |
