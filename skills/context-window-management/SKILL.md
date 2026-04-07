---
name: context-window-management
capability: Manage what information is loaded into the agent's context to maximize relevance and minimize noise.
---

# Context Window Management

## What This Skill Enables

Agents with this skill can work effectively within their finite context window by making deliberate choices about what to load, when to load it, and what to leave out. Without it, agents either overload their context with irrelevant files (leading to confused reasoning and missed details) or underload it (leading to hallucinated APIs and wrong assumptions). Skilled context management is the difference between an agent that stays sharp on task and one that degrades as complexity grows.

## Core Competencies

### 1. Understand the Context Budget

Think of the context window as a fixed budget measured in tokens. Every file, every message, every tool output consumes part of that budget. Once exhausted, older information is lost or the agent must start a new session.

| Budget Category | Priority | Guidance |
|---|---|---|
| **Task specification** | Always load | The issue, ticket, or user request that defines what "done" looks like. |
| **Directly relevant source files** | Always load | Files you are about to read or edit. |
| **Related test files** | Load early | Tests define expected behavior and catch mistakes. |
| **Config and schema files** | Load when needed | `tsconfig.json`, `.env.example`, database schemas -- load when the task touches them. |
| **Framework boilerplate** | Avoid | Large generated files, lockfiles, and vendor directories are noise. |
| **Unrelated modules** | Avoid | Code in other features or services that the current task does not touch. |

Rule of thumb: if a file will not influence a decision you make in the next few steps, do not load it yet.

### 2. Choose What to Load vs. What to Omit

Before reading any file, ask: "Will this change what I do next?"

**Load:**
- The file(s) you are about to modify.
- The test file(s) that cover those modules.
- The specification or issue body for the current task.
- Type definitions or interfaces that the modified code depends on.
- Configuration files only when the task involves config changes.

**Omit:**
- Files in unrelated features or services.
- Large generated files (`package-lock.json`, `yarn.lock`, migration dumps).
- Entire directories when you only need one file from them.
- Documentation that restates what the code already shows.
- Previous conversation turns that are no longer relevant (in multi-turn sessions).

### 3. Apply Progressive Disclosure

Do not load everything at the start. Instead, load information in layers:

```
Layer 1 -- Orientation (start of task)
  Read the task spec or issue body.
  Read a directory listing or file tree to understand project structure.
  Read the primary file you will modify (just the relevant section if large).

Layer 2 -- Investigation (as questions arise)
  Read type definitions or interfaces when you encounter unfamiliar types.
  Read test files when you need to understand expected behavior.
  Read config files when the task requires configuration changes.

Layer 3 -- Verification (before finalizing)
  Re-read the spec to confirm you addressed every requirement.
  Read test output to confirm all tests pass.
  Read adjacent code only if integration concerns arise.
```

This layered approach keeps the context focused on what matters at each phase.

### 4. Decide When to Re-Read vs. Trust Cached Context

Files loaded earlier in the conversation may become stale if you or another process modified them. Use this decision framework:

| Situation | Action |
|---|---|
| You just edited the file in a previous step. | Trust your edit -- you know what changed. |
| Another tool or process modified the file. | Re-read it. You do not know what changed. |
| The file was loaded more than 20 turns ago. | Re-read it. Your memory of details may be inaccurate. |
| You need to quote exact syntax (imports, function signatures). | Re-read it. Do not guess at spelling or argument order. |
| You only need a high-level fact ("does this file use TypeScript?"). | Trust cached context. Re-reading is wasteful for coarse facts. |

When in doubt, re-read. A redundant read costs tokens but prevents hallucination. A hallucinated import path costs debugging time.

### 5. Recognize Signs of Context Overload

When the context becomes too full, agent behavior degrades in predictable ways:

| Symptom | What Is Happening | Correction |
|---|---|---|
| Repeating information already stated. | Earlier turns are being pushed out of effective attention. | Summarize progress so far in a compact message; shed old detail. |
| Confusing details from two different files. | Too many files are loaded and blending together. | Close unrelated files. Work on one module at a time. |
| Forgetting a requirement from the spec. | The spec was loaded too early and is now far from attention. | Re-read the spec before finalizing. |
| Generating code that contradicts a type definition loaded earlier. | The definition has scrolled out of effective range. | Re-read the type definition immediately before writing code that depends on it. |
| Producing increasingly vague or generic responses. | Total context volume is overwhelming focused reasoning. | Start a new session with a compact summary of progress and only the essential files. |

## Behavioral Rules

1. Never load a file without a specific reason tied to the current or next step of the task.
2. Always load the task specification and the primary file to be modified before writing any code.
3. Prefer reading targeted sections of large files over loading entire files when only a few lines are relevant.
4. Re-read any file you need to quote exactly, rather than relying on memory from earlier in the conversation.
5. When context feels overloaded, summarize progress and shed stale information before continuing.

## Failure Modes

| Failure | Symptom | Correction |
|---|---|---|
| Greedy loading | Agent reads every file in the directory "just in case." | Load only what is needed for the immediate next step. |
| Stale context | Agent references an old version of a file it has since modified. | Re-read files after modification by other tools or processes. |
| Hallucinated details | Agent invents function signatures or config keys not in any loaded file. | Re-read the source of truth before generating dependent code. |
| Spec drift | Agent forgets a requirement because the spec was loaded 50 turns ago. | Re-read the spec at the start of each major phase of the task. |
| Context hoarding | Agent refuses to start a new session despite clear degradation. | Recognize overload symptoms; summarize and restart with a clean context. |
