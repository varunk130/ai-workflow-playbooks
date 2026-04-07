---
name: error-interpretation
capability: Read error messages, stack traces, and logs systematically rather than guessing at fixes.
---

# Error Interpretation

## What This Skill Enables

Agents with this skill can diagnose failures methodically by reading what the system is actually telling them. Without it, agents fall into shotgun debugging -- trying random fixes, changing multiple things at once, and hoping something sticks. A skilled agent reads the error, forms a hypothesis, and verifies it before writing a single line of corrective code.

## Core Competencies

### 1. Read the Actual Error Message First

This sounds obvious but is the most commonly skipped step. Before doing anything:

1. Copy the **full** error output, not just the first line.
2. Identify the **error type or code** (e.g., `TypeError`, `ECONNREFUSED`, `HTTP 403`, exit code 137).
3. Identify the **error message** (the human-readable explanation after the type).
4. Note any **file path and line number** referenced.

```
# Example: Don't just see "TypeError". Read the whole thing.

TypeError: Cannot read properties of undefined (reading 'map')
    at renderList (src/components/UserList.tsx:14:22)
    at processChild (node_modules/react-dom/cjs/react-dom.development.js:1234:12)

# What to extract:
#   Type:    TypeError
#   Message: Cannot read properties of undefined (reading 'map')
#   Where:   src/components/UserList.tsx, line 14, column 22
#   What:    Something is undefined when .map() is called on it
```

### 2. Parse Stack Traces Correctly

Stack traces list frames from most recent call to oldest. Your job is to find the boundary between your code and framework/library code.

| Frame Type | How to Identify | What to Do |
|---|---|---|
| **Your code** | File path is in `src/`, `app/`, `lib/`, or your project directories. | This is where the bug lives. Start here. |
| **Framework code** | Path is in `node_modules/`, standard library, or runtime internals. | Usually not the bug. Tells you what the framework was doing when your code failed. |
| **Generated code** | Path references `.next/`, `dist/`, `build/`, `__pycache__/`. | Map back to the source file. Don't edit generated code. |

Read from the top. The first frame in **your** code is almost always the place to investigate.

### 3. Categorize the Error

Different categories demand different investigation strategies:

| Category | Examples | First Move |
|---|---|---|
| **Syntax** | `SyntaxError`, `IndentationError`, unexpected token | Go to the exact line. Look for typos, missing brackets, bad indentation. |
| **Type** | `TypeError`, `AttributeError`, `ClassCastException` | Check what value is actually present vs. what was expected. Add a log or breakpoint one line above. |
| **Runtime** | `RangeError`, `IndexError`, `NullPointerException` | Trace the data flow. Where did the bad value originate? |
| **Network** | `ECONNREFUSED`, `TimeoutError`, HTTP 4xx/5xx | Check if the target service is running. Verify URL, port, auth. |
| **Permission** | `EACCES`, `PermissionError`, HTTP 401/403 | Check file permissions, API keys, IAM roles, token expiry. |
| **Resource** | `ENOMEM`, exit code 137, `OOMKilled` | The process ran out of memory or disk. This is an environment issue, not a code bug. |

### 4. Avoid Shotgun Debugging

Shotgun debugging is changing things at random until the error disappears. It wastes time and often introduces new bugs. Instead, follow this discipline:

```
1. Read the error message (competency 1).
2. Locate the failing line (competency 2).
3. Form ONE hypothesis about what is wrong.
4. Identify ONE change that would confirm or refute the hypothesis.
5. Make that change and re-run.
6. If the hypothesis was wrong, revert and form a new one.
```

Never make two changes at once. If you change two things and the error goes away, you do not know which change fixed it -- and one of them may have introduced a latent bug.

### 5. Use Error Messages to Form Hypotheses

Translate the error into a plain-language hypothesis before writing any fix:

| Error Message | Hypothesis |
|---|---|
| `Cannot read properties of undefined (reading 'map')` | The variable I'm calling `.map()` on is `undefined`. It was probably not initialized, or the API returned an unexpected shape. |
| `ECONNREFUSED 127.0.0.1:5432` | The PostgreSQL server is not running on localhost, or it is on a different port. |
| `Module not found: Can't resolve './utils'` | The file `utils` does not exist at that relative path, or the extension is wrong, or the file was moved. |
| `403 Forbidden` | The credentials are missing, expired, or lack the required scope for this endpoint. |

A hypothesis is testable. "Something is broken" is not a hypothesis. "The `users` prop is undefined because the parent component does not pass it" is.

## Behavioral Rules

1. Never propose a fix without first quoting the exact error message and explaining what it means.
2. Never change more than one thing at a time when diagnosing an error.
3. Always identify whether the failing line is in your code, framework code, or generated code before investigating.
4. Revert any diagnostic change that did not help before trying the next hypothesis.
5. If the same error persists after three hypotheses, step back and re-read the full output from scratch -- you likely missed something.

## Failure Modes

| Failure | Symptom | Correction |
|---|---|---|
| Shotgun debugging | Multiple unrelated changes in one attempt; "let me try this" without rationale. | One hypothesis, one change, one test run. Revert if wrong. |
| Ignoring the message | Agent says "I see an error" but never quotes or analyzes it. | Always paste and parse the full error before acting. |
| Fixing the wrong frame | Agent edits framework or generated code instead of source. | Trace the stack to the first frame in project source code. |
| Treating symptoms | Error goes away but root cause remains (e.g., wrapping in try/catch without handling). | Ensure the fix addresses why the error occurs, not just that it occurs. |
| Stale hypothesis | Agent keeps pursuing the same theory despite contradicting evidence. | After two failed attempts on one theory, discard it and re-read the error. |
