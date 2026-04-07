---
name: requirement-extraction
capability: Transform vague user inputs into structured, actionable requirements with explicit assumptions.
---

# Requirement Extraction

## What This Skill Enables
An agent that can take ambiguous human input — "make the dashboard faster," "add authentication," "fix the search" — and systematically convert it into concrete, testable requirements before writing code. Agents without this skill build the wrong thing confidently.

## Core Competencies

### 1. Signal Capture
Receive input without interpreting or filtering:
- Record the exact words the user used
- Note what was said AND what was left unsaid
- Identify the stated goal vs the implied constraints
- Distinguish between the problem description and a proposed solution

### 2. Ambiguity Detection
Flag vague language that needs clarification:

| Vague Input | What's Missing | Clarifying Question |
|-------------|---------------|-------------------|
| "Make it faster" | Faster by how much? Which operation? | "What's the current latency, and what's the target?" |
| "Add user roles" | Which roles? What permissions? | "What actions should each role be able to perform?" |
| "Fix the bug" | Which bug? How to reproduce? | "Can you describe the steps to trigger this behavior?" |
| "It should be secure" | Against what threats? What data? | "What sensitive data does this handle, and who should access it?" |
| "Like Spotify does it" | Which specific feature/behavior? | "Can you describe the specific interaction you're referring to?" |

### 3. Assumption Surfacing
Make every implicit assumption explicit before proceeding:

```
STATED: "Add a login page"

ASSUMPTIONS I'M MAKING:
- Email + password authentication (not OAuth/SSO)
- Single user type (no admin vs regular distinction)
- Session-based auth (not token-based)
- Standard form with email and password fields
- Redirect to home page after successful login
- Show error message for invalid credentials
- No "remember me" or "forgot password" in v1

RISKS IF WRONG:
- OAuth requirement would change the entire auth architecture
- Multiple user types affect the data model and routing

→ Please confirm or correct these before I proceed.
```

### 4. Requirement Structuring
Convert confirmed requirements into testable statements:

**Format**: `WHEN [condition], the system MUST/SHOULD/MAY [behavior]`

Examples:
- `WHEN a user submits valid credentials, the system MUST create a session and redirect to /dashboard`
- `WHEN a user submits invalid credentials, the system MUST display "Invalid email or password" without revealing which field was wrong`
- `WHEN a session expires, the system SHOULD redirect to /login with a "session expired" message`
- `WHEN a logged-in user visits /login, the system MAY redirect to /dashboard`

### 5. Scope Fencing
Define what's in and what's out:

```
IN SCOPE:
- Login form with email/password
- Session creation and validation
- Logout endpoint
- Protected route middleware

OUT OF SCOPE (for this iteration):
- Registration / signup
- Password reset
- OAuth / social login
- Multi-factor authentication
- Remember me functionality
```

## Behavioral Rules

1. **Never assume silently** — If you fill in a gap, say so and ask for confirmation
2. **Ask before building** — One clarifying question saves hours of rework
3. **Separate problem from solution** — "We need Redis" is a solution; "responses are slow" is the problem
4. **Quantify wherever possible** — "Fast" means nothing; "<200ms p95 response time" is testable
5. **Default conservatively** — When in doubt, choose the simpler interpretation and note it as an assumption

## Failure Modes

| Failure | Symptom | Correction |
|---------|---------|------------|
| Assuming requirements | Building features the user didn't ask for | Surface every assumption explicitly |
| Gold-plating | Adding OAuth when they asked for basic login | Stick to stated scope; suggest enhancements separately |
| Not asking follow-ups | Implementing based on first message alone | Probe for edge cases, error states, and constraints |
| Treating solutions as requirements | "Use Redis" → implementing Redis without understanding the actual need | Ask "what problem does this solve?" first |
