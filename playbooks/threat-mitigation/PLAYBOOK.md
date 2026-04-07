---
name: threat-mitigation
trigger: when code touches authentication, authorization, user input handling, database queries, secret storage, HTTP headers, or dependency management
---

# Threat Mitigation Playbook

## Objective

Systematically harden every security-relevant code path before it ships. This playbook ensures the agent applies defense-in-depth across three trust boundaries rather than relying on a single perimeter check.

## Triggers

Activate this playbook when any of the following appear in the task:

- Authentication or authorization logic is added or modified.
- User-supplied data flows into queries, templates, or shell commands.
- Secrets, tokens, or API keys are referenced in source files.
- New dependencies are introduced to the project.
- HTTP response headers or CORS policies are configured.
- A reviewer or issue explicitly flags a security concern.

## OWASP Top 10 Awareness

Before writing any security-sensitive code, recall the current OWASP Top 10 categories. The most frequently relevant to agent-generated code are:

1. **Broken Access Control** -- verify every endpoint enforces authorization.
2. **Injection** -- parameterize all queries; never concatenate user input into commands.
3. **Cryptographic Failures** -- use strong hashing (bcrypt/argon2); enforce TLS.
4. **Security Misconfiguration** -- ship strict defaults; disable debug modes in production.
5. **Vulnerable Components** -- audit dependencies before adding them.

Keep this list in working memory for every step below.

## Three-Boundary Model

All data in the system crosses one or more trust boundaries. Validate at every crossing.

| Boundary | Description | Key Controls |
|---|---|---|
| **User Input Boundary** | Data arriving from browsers, mobile clients, or external APIs | Schema validation, allowlists, length limits, encoding |
| **Service-to-Service Boundary** | Calls between internal microservices or third-party APIs | Mutual TLS, signed tokens, request authentication, rate limiting |
| **Data Storage Boundary** | Reads and writes to databases, caches, or file systems | Parameterized queries, encrypted-at-rest, least-privilege DB roles |

## Workflow

### Step 1 -- Map the Attack Surface

Identify every point where untrusted data enters the change. List each entry point, the boundary it crosses, and the downstream consumer.

### Step 2 -- Validate All Input

Apply input validation at the User Input Boundary:

- **Allowlists over denylists.** Define what IS permitted, not what is forbidden.
- **Type checking.** Parse inputs into expected types (integer, UUID, enum) immediately.
- **Length limits.** Enforce maximum lengths before any processing occurs.
- **Encoding.** Context-encode output (HTML-encode for templates, URL-encode for query strings).

### Step 3 -- Harden Authentication and Sessions

- Hash passwords with bcrypt (cost >= 12) or argon2id. Never use MD5 or SHA-256 alone.
- Generate tokens with a CSPRNG; minimum 256 bits of entropy.
- Set session cookies with `HttpOnly`, `Secure`, `SameSite=Strict`.
- Implement token expiration and rotation. Short-lived access tokens; longer-lived refresh tokens stored securely.
- Enforce account lockout or progressive delays after repeated failed attempts.

### Step 4 -- Secure Service-to-Service Communication

- Authenticate every internal call with signed JWTs or mutual TLS.
- Validate the audience (`aud`) and issuer (`iss`) claims on every received token.
- Apply rate limiting and circuit breakers to prevent cascade abuse.

### Step 5 -- Protect the Data Storage Boundary

- Use parameterized queries or an ORM. Zero raw string concatenation in SQL.
- Apply the principle of least privilege: the application DB user should only have SELECT/INSERT/UPDATE on the tables it needs.
- Encrypt sensitive columns at rest. Use envelope encryption where the platform supports it.

### Step 6 -- Manage Secrets Properly

- **Never commit secrets to source control.** Not even in "example" files.
- Load secrets from environment variables or a vault service (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault).
- Rotate secrets on a schedule and immediately after any suspected leak.
- Add patterns to `.gitignore` and set up pre-commit hooks (e.g., `git-secrets`) to block accidental commits.

### Step 7 -- Audit Dependencies

- Run `npm audit` (or the equivalent for the project's package manager) before adding any dependency.
- Check the license of every new package for compatibility.
- Prefer packages with active maintenance, a security policy, and a small dependency tree.
- Pin dependency versions and use lockfiles. Review lockfile diffs in every PR.

### Step 8 -- Set Security Headers

Ensure the application returns these HTTP headers on every response:

| Header | Recommended Value |
|---|---|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'` (adjust per app) |
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains; preload` |
| `X-Frame-Options` | `DENY` |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` |

### Step 9 -- Verify and Test

- Write at least one test per boundary that submits malicious input and asserts rejection.
- Run SAST/DAST tools if available in CI. Confirm zero new high-severity findings.
- Manually trace one end-to-end request through all three boundaries to confirm controls are active.

## Guardrails

- Never disable security controls to make a test pass. Fix the test.
- Never store passwords in plaintext or reversible encryption.
- Never trust client-side validation as the only layer of defense.
- Never log secrets, tokens, or full credit card numbers.
- Every SQL query must use parameterized statements. No exceptions.

## Exit Criteria

- [ ] All three trust boundaries have explicit validation in place.
- [ ] Secrets are loaded from environment or vault, with no hard-coded values in source.
- [ ] `npm audit` (or equivalent) reports zero high/critical vulnerabilities.
- [ ] Security headers are set and verified in an integration test or manual check.
- [ ] Auth flows use strong hashing, secure cookies, and token expiration.
- [ ] At least one negative test exists per boundary (malicious input is rejected).

## Anti-Patterns

| Anti-Pattern | Why It Fails | Correct Approach |
|---|---|---|
| Denylist-based input filtering | Attackers find encodings or variants you missed | Use allowlists that define exactly what is accepted |
| Storing secrets in `.env` committed to repo | Secrets end up in git history permanently | Use `.env.example` with placeholders; load real values from vault |
| Rolling your own encryption or hashing | Subtle flaws lead to broken crypto | Use well-audited libraries (bcrypt, argon2, libsodium) |
| Disabling HTTPS in development | Dev/prod parity breaks; bugs hide | Use self-signed certs or mkcert for local TLS |
| Skipping dependency audit for "small" packages | Small packages can carry large vulnerabilities | Audit every addition regardless of size |
| Trusting internal services without auth | Lateral movement becomes trivial after any breach | Authenticate every service-to-service call |
| Adding `'unsafe-eval'` to CSP to fix a bug | Reopens XSS attack surface | Refactor code to remove eval; use nonce-based CSP if dynamic scripts are required |
