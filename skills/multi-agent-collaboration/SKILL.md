---
name: multi-agent-collaboration
capability: Coordinate effectively when multiple AI agents work on the same codebase or feature.
---

# Multi-Agent Collaboration

## What This Skill Enables
An agent that can operate alongside other agents — dividing work, avoiding conflicts, and maintaining consistency — without stepping on each other's changes or duplicating effort. Without this skill, multi-agent workflows produce merge conflicts, inconsistent patterns, and wasted cycles.

## Core Competencies

### 1. Work Partitioning
Divide tasks so agents work on independent surfaces:
- **By file**: Agent A owns `auth/`, Agent B owns `dashboard/`
- **By layer**: Agent A handles API endpoints, Agent B handles UI components
- **By feature slice**: Agent A builds user creation end-to-end, Agent B builds search end-to-end
- **Never**: Two agents modifying the same file simultaneously

### 2. Contract Agreement
When agents produce code that connects, define the contract first:
- Agree on interface signatures (function names, parameter types, return types)
- Agree on data shapes (request/response schemas, event payloads)
- Write the interface or type definition first, implement second
- Both agents can work in parallel once the contract is locked

### 3. Convention Adherence
Multiple agents must produce code that looks like one person wrote it:
- Read the project's style guide or infer it from existing code before writing
- Use the same naming patterns, file structure, and error handling approach
- Run the same linter and formatter configuration
- Follow the same commit message format

### 4. Conflict Prevention
Avoid merge conflicts proactively:
- Check which files other agents have modified before starting
- Prefer additive changes (new files) over modifications to shared files
- When shared files must change, coordinate the order — one agent goes first
- Use feature flags to integrate work independently

### 5. Handoff Protocol
When passing work to another agent:
```
HANDOFF NOTE:
- What I completed: [list of files/features implemented]
- What I tested: [which tests pass, what was verified]
- What remains: [tasks not yet started]
- Assumptions I made: [decisions the next agent should know about]
- Known issues: [anything broken or incomplete]
- Files to read first: [most important context files]
```

## Behavioral Rules

1. **Own your scope** — Never modify files outside your assigned area without coordination
2. **Contracts before code** — When work connects, agree on the interface first
3. **Commit frequently** — Small, frequent commits reduce merge conflict surface area
4. **Document decisions** — Other agents can't read your reasoning; write it down
5. **Verify integration** — After merging parallel work, run the full test suite

## Failure Modes

| Failure | Symptom | Correction |
|---------|---------|------------|
| Both agents edit the same file | Merge conflicts, lost changes | Partition by file or coordinate editing order |
| Inconsistent patterns | Code looks like two different projects | Align on conventions before starting |
| Missing handoff context | Next agent repeats work or breaks assumptions | Use the handoff protocol template |
| No integration testing | Individual pieces work, combined system doesn't | Run full suite after merging any parallel work |
