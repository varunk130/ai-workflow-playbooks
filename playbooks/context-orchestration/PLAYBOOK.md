---
name: Context Orchestration
trigger: When setting up, tuning, or debugging the information flow between your project and an AI coding agent
---

# Context Orchestration

## Objective

Feed the AI agent the **right context, at the right time, in the right amount** so that its
output is accurate, relevant, and grounded in your actual codebase. Context quality is the
single largest lever you have over agent output quality — a perfectly prompted agent with
poor context will consistently underperform a simply prompted agent with excellent context.

## Triggers — Use This Playbook When

- You are onboarding a new AI coding agent to an existing repository.
- Agent responses are generic, hallucinated, or ignore project conventions.
- You are configuring rules files (`.claude.md`, `.cursor/rules`, `.github/copilot-instructions.md`).
- You are wiring up MCP servers to give the agent live access to docs, APIs, or databases.
- You notice the agent "forgetting" decisions made earlier in the same session.
- Token usage is spiking without a corresponding increase in output quality.

## Do Not Use When

- The problem is a malformed prompt, not missing context (see the Prompt Engineering playbook).
- You need to restrict agent capabilities for safety — that is an access-control concern, not context.
- The agent already produces correct output; adding more context adds cost with no benefit.

## Workflow

### 1. Audit What the Agent Knows Today

Before adding context, map what the agent already receives automatically:
- **Implicit context**: the file currently open, recent edits, terminal output.
- **Rules files**: `.claude.md` (Claude Code), `.cursor/rules/*.mdc` (Cursor), `.github/copilot-instructions.md` (Copilot).
- **MCP integrations**: any Model Context Protocol servers already connected (filesystem, GitHub, databases, documentation search).
- **Session history**: prior turns in the current conversation.

Run a quick test: ask the agent a question whose answer requires project-specific knowledge
(e.g., "What ORM do we use?" or "Where is the auth middleware?"). If it guesses wrong or
hedges, that knowledge is missing from its context.

### 2. Design Layered Context

Organize context into three tiers and keep each one focused:

| Layer | Scope | Typical vehicle | Refresh cadence |
|---|---|---|---|
| **Project-wide** | Applies to every task in the repo | Rules files, `CLAUDE.md` at repo root | Changes with major architecture decisions |
| **Task-specific** | Applies to the current unit of work | Files added to context, MCP query results, inline instructions | Each new task or PR |
| **Session-specific** | Applies only to this conversation | Prior turns, scratchpad notes, `/compact` summaries | Resets when the session ends |

**Project-wide context** should include: tech stack, directory structure conventions, naming
standards, test expectations, forbidden patterns, and links to canonical docs. Keep it under
80 lines — agents lose signal in long preambles.

**Task-specific context** should include: the exact files being modified, relevant type
definitions or interfaces, the failing test or error message, and the acceptance criteria.

**Session-specific context** is what the agent accumulates turn-by-turn. Monitor its growth;
when it becomes noisy, summarize or start a fresh session.

### 3. Configure Rules Files

Rules files are your most reliable context channel because they load automatically.

- **Claude Code**: place a `CLAUDE.md` at the repo root for global rules; add `CLAUDE.md`
  files in subdirectories for scoped rules that apply only when working in that subtree.
- **Cursor**: create `.cursor/rules/*.mdc` files with frontmatter specifying `globs` for
  auto-attachment or `alwaysApply: true` for global rules.
- **Copilot**: use `.github/copilot-instructions.md` for repository-level instructions.

Write rules files in short, imperative sentences. State what to do, not what to avoid (unless
the anti-pattern is common enough to warrant an explicit ban). Version-control these files
and review changes in PRs just like production code.

### 4. Wire Up MCP Integrations for Dynamic Context

Static rules files cannot cover everything. Use MCP servers to give the agent live access to:
- **Documentation search** (e.g., `microsoft-learn`, custom doc servers) so it retrieves
  current API references instead of hallucinating from training data.
- **GitHub** (issues, PRs, code search) so it can look up related changes and discussions.
- **Databases or internal APIs** for schema introspection or runtime configuration.
- **Figma or design tools** for translating designs into code with accurate specs.

Register MCP servers in `.claude/settings.json`, `.cursor/mcp.json`, or the equivalent for
your tooling. Test each integration by asking the agent a question only that data source
can answer.

### 5. Manage the Context Window

Every token of context you add displaces a token the agent could use for reasoning. Be
deliberate about what goes in:

- **Include**: files being edited, their direct dependencies, relevant type signatures,
  the specific error or test output, and the acceptance criteria for the task.
- **Omit**: unrelated files, large generated artifacts (lock files, build output, migration
  dumps), verbose logs beyond the relevant excerpt, and entire files when only a few lines
  are needed.
- **Re-read files** when the agent has made changes since they were first loaded, or when
  more than ~15 turns have elapsed. Do not assume cached context is still accurate.
- **Use `/compact`** or equivalent summarization when session history grows large but you
  want to preserve key decisions without the full transcript.

### 6. Implement Memory and Persistent Context

For decisions that should survive across sessions:
- Write them into the project-wide rules file so every future session inherits them.
- Use a `CONVENTIONS.md` or architecture-decision-record (ADR) file that the agent can
  be instructed to read at the start of relevant tasks.
- Some agents support memory files (e.g., Cursor's `memory.md`); populate these with
  outcomes, not raw conversation logs.

Persistent context should be **curated, not appended**. Periodically review memory files
and prune entries that are no longer relevant.

### 7. Validate and Iterate

After configuring context, re-run the audit from Step 1. Ask the same project-specific
questions. If the agent now answers correctly and follows conventions without being
reminded, your context orchestration is working. If not, identify the gap and add the
minimum context needed to close it.

## Guardrails

- **Never put secrets** (API keys, credentials, connection strings) in rules files or context that gets sent to a hosted model.
- **Cap project-wide rules** at roughly 80 lines. If you need more, split into scoped, directory-level files.
- **Review context costs**: monitor token usage before and after changes. A 2x increase in input tokens should produce a measurable improvement in output quality.
- **Do not rely on session memory for critical decisions**. If a decision matters, write it into a persistent file.
- **Test rules files in CI** if your tooling supports it — a malformed rules file silently degrades every interaction.

## Exit Criteria

- [ ] Agent correctly answers project-specific questions without hedging or hallucinating.
- [ ] Rules files exist, are version-controlled, and are under 80 lines each.
- [ ] MCP integrations are tested and confirmed working for each data source.
- [ ] Context window usage is intentional: no large files included without reason.
- [ ] Persistent decisions are recorded in rules files or ADRs, not left in session history.
- [ ] Token usage is stable and proportional to task complexity.

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| **Context overload** — dumping entire repo into the context window | Drowns the signal in noise; agent loses focus and misses key details | Include only the files and sections relevant to the current task |
| **Stale context** — relying on a file the agent read 20 turns ago that has since changed | Agent generates code against an outdated version of the file | Re-read files after modifications or long gaps between references |
| **Assuming context persists** — expecting the agent to remember a prior session | Most agents start fresh each session; prior decisions are lost | Write important decisions into rules files or persistent docs |
| **Monolithic rules file** — a 300-line rules file covering every edge case | Agent attention dilutes across too many instructions; contradictions creep in | Split into project-wide (global) and directory-scoped (local) files |
| **No context at all** — relying on the agent's training data for project specifics | Agent invents plausible but wrong conventions, dependencies, or file paths | Provide explicit rules and load relevant files into context |
| **Verbose error pasting** — copying 500 lines of stack trace when 10 lines suffice | Wastes tokens and buries the actual error | Trim to the relevant frames and the root-cause message |
| **MCP without validation** — connecting an MCP server and assuming it works | Silent failures mean the agent falls back to guessing | Test each MCP tool with a targeted question after setup |
