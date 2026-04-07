---
name: codebase-navigation
capability: Efficiently explore, map, and understand unfamiliar codebases before making changes.
---

# Codebase Navigation

## What This Skill Enables
An agent that can quickly orient itself in any codebase — finding the right files, understanding the architecture, and identifying conventions — before writing a single line of code. Agents without this skill guess at file locations, miss existing utilities, and create duplicates.

## Core Competencies

### 1. Project Reconnaissance
Before touching code, build a mental map:
- Read `README.md`, `CONTRIBUTING.md`, and any project-level config files
- Identify the package manager and build system (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`)
- Map the directory structure — where does source live? Tests? Config? Docs?
- Identify the entry point(s) of the application

### 2. Convention Detection
Infer the project's conventions from existing code:
- **Naming**: camelCase vs snake_case vs kebab-case — match what exists
- **File organization**: feature-based vs layer-based vs hybrid
- **Import style**: relative vs absolute, barrel files, path aliases
- **Testing conventions**: co-located tests vs separate `__tests__` directory, naming patterns
- **Error handling**: custom error classes, result types, try/catch patterns

### 3. Dependency Mapping
Trace how components connect:
- Follow imports from the entry point to understand the call graph
- Identify shared utilities and where they live
- Locate configuration and environment variable usage
- Map database models/schemas to their consumers
- Identify external API integrations and their client modules

### 4. Search Strategies
Use the right tool for each search type:

| Goal | Strategy |
|------|----------|
| Find a file by name | Glob patterns: `**/UserService.*`, `**/*.config.*` |
| Find where a function is defined | Grep for `function name(` or `def name` or `class Name` |
| Find where a function is called | Grep for the function name, exclude the definition file |
| Understand data flow | Start at the API endpoint, follow the handler chain inward |
| Find related tests | Look for files matching `*.test.*`, `*.spec.*`, or `test_*` near the source |

### 5. Architecture Recognition
Identify common architectural patterns:
- **MVC/MVVM**: Models, views/templates, controllers/viewmodels in separate directories
- **Clean Architecture**: Domain/entities at the core, use cases, adapters, infrastructure at edges
- **Microservices**: Multiple `services/` or separate packages with their own entry points
- **Monorepo**: `packages/` or `apps/` directory with shared libraries
- **Serverless**: Function handlers in `functions/` or `lambdas/`, infrastructure as code nearby

## Behavioral Rules

1. **Read before writing** — Never create a file without first searching for an existing one that does the same thing
2. **Match conventions** — New code should be indistinguishable from existing code in style
3. **Minimize exploration scope** — Start narrow (the specific module), widen only when needed
4. **Document what you find** — If the architecture isn't documented, note your findings for context
5. **Respect boundaries** — If the project separates concerns into layers, don't bypass them

## Failure Modes

| Failure | Symptom | Correction |
|---------|---------|------------|
| Skipping reconnaissance | Creating files in wrong locations | Always map the project before writing |
| Ignoring conventions | Code looks foreign in the PR diff | Read 3-5 existing files in the same area first |
| Shallow search | Missing existing utilities, creating duplicates | Search by concept, not just by exact name |
| Assuming structure | "I expect a `utils/` folder" when the project uses `lib/shared/` | Let the project tell you its structure |
