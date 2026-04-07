---
name: contract-first-apis
trigger: Use when designing or modifying APIs, service interfaces, or data contracts. Activate before implementing any endpoint.
---

# Contract-First APIs

## Objective
Design APIs starting from the contract (schema, types, response shapes) before writing implementation code. A well-defined contract protects both producers and consumers from breaking changes and enables parallel development.

## Triggers
- Creating a new API endpoint or service interface
- Modifying an existing API that has consumers
- Integrating with an external service
- Defining communication between frontend and backend

## Workflow

### Step 1: Define the Contract
Before any implementation, document:

```
ENDPOINT: [HTTP method] [path]
PURPOSE: [What this endpoint does — one sentence]
REQUEST:
  Headers: [required headers]
  Params: [path and query parameters with types]
  Body: [request body schema with types and constraints]
RESPONSE:
  Success (200/201): [response body schema]
  Validation Error (400): [error response shape]
  Not Found (404): [error response shape]
  Server Error (500): [error response shape]
SIDE EFFECTS: [what this endpoint changes in the system]
IDEMPOTENCY: [is this safe to retry?]
```

### Step 2: Apply Naming Conventions
- Resources are **nouns**, not verbs: `/users`, not `/getUsers`
- Use plural nouns for collections: `/orders`, `/items`
- Nest resources to show relationships: `/users/{id}/orders`
- Use query parameters for filtering, sorting, pagination: `?status=active&sort=created_at&limit=20`
- Versioning in the path or header: `/v1/users` or `Accept: application/vnd.api+json;version=1`

### Step 3: Design Error Responses Consistently
All errors follow the same shape:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Human-readable description",
    "details": [
      { "field": "email", "issue": "must be a valid email address" }
    ]
  }
}
```

Rules:
- Error codes are stable machine-readable identifiers — never change them
- Error messages are human-readable and safe to display
- Details provide field-level context when applicable
- Never expose stack traces, internal paths, or SQL in error responses

### Step 4: Handle Versioning and Evolution
- **Additive changes are safe**: New optional fields, new endpoints, new enum values
- **Breaking changes require versioning**: Removing fields, changing types, renaming keys
- **Observable behavior is the contract**: If consumers depend on response order, timing, or undocumented fields, those become part of the contract whether intended or not
- Document deprecation timelines when evolving APIs

### Step 5: Implement Boundary Validation
At every API boundary:
1. Validate input against the contract schema (types, ranges, formats)
2. Reject invalid requests early with descriptive error responses
3. Sanitize inputs to prevent injection attacks
4. Apply rate limiting and request size limits
5. Log validation failures for monitoring

### Step 6: Write Contract Tests
For each endpoint, write tests that verify:
- Valid requests produce expected responses
- Missing required fields return 400 with the correct error shape
- Invalid types return 400 with field-level details
- Authentication failures return 401
- Authorization failures return 403
- Not-found cases return 404
- The response body matches the documented schema exactly

## Guardrails
- Never implement an endpoint before its contract is documented
- Do not return different error shapes from different endpoints — consistency is mandatory
- Internal implementation errors must not leak through the API surface
- Pagination is required for any endpoint that can return unbounded lists
- All timestamps use ISO 8601 format in UTC

## Exit Criteria
- [ ] Every endpoint has a documented contract (request/response/error schemas)
- [ ] Error responses follow the standard shape across all endpoints
- [ ] Boundary validation rejects invalid input with descriptive errors
- [ ] Contract tests verify request/response shapes for all status codes
- [ ] Breaking changes are versioned; additive changes are documented
- [ ] No internal implementation details leak through API responses

## Anti-Patterns

| Shortcut | Why It Fails |
|----------|-------------|
| Implementing the endpoint first, documenting later | The contract drifts from the docs; consumers build against undocumented behavior |
| Different error shapes per endpoint | Consumers must special-case every error handler |
| Returning 200 for error conditions | Masks failures from monitoring and client error handling |
| Exposing database IDs or internal field names | Couples consumers to your storage layer |
| No pagination on list endpoints | Response size grows unbounded; latency spikes in production |
| "We'll version it when we need to" | By then, consumers depend on the current shape and migration is painful |
