# Token-Budget Guidance per Playbook Stage

Operational guidance for budgeting cost and tokens across the playbook lifecycle. Designed to give teams running these playbooks a defensible answer to *"how much will this cost per use, and where do we cut if the budget bites?"*

This guide does **not** prescribe specific dollar figures (model prices change too fast). It prescribes a **structure** for budgeting and a set of stage-specific patterns for cost reduction that hold across model generations.

---

## Why Token Budgets Matter for Playbooks

A playbook is a *repeated* workflow. A 10× cost regression in a one-off prompt is a one-time annoyance; a 10× regression in a playbook used 200×/month is a budget event. Token budgeting at the playbook stage is the highest-leverage place to instrument cost.

The four levers that matter, in order of impact:

| Lever | Typical Impact | Where in the Playbook |
|-------|---------------:|------------------------|
| **Context size** | 5-20× | Discover, Plan stages (research dumps, doc grounding) |
| **Model selection** | 3-10× | Every stage - match model to task complexity |
| **Reasoning verbosity** | 2-4× | Plan, Build stages (reasoning models, verbose chain-of-thought) |
| **Caching strategy** | 2-5× | Iterative stages (Build, Refine) - same context, many turns |

---

## The Per-Stage Budget Pattern

Every playbook stage gets four budget annotations:

| Annotation | What It Captures |
|------------|------------------|
| **Expected Input Tokens** | Typical context size at this stage (range, not a point estimate) |
| **Expected Output Tokens** | Typical output length |
| **Recommended Model Tier** | Cheap / Standard / Premium - based on task complexity |
| **Cost Reduction Levers** | The 1-3 things to do *first* if this stage's cost overruns |

The annotations live alongside the stage definition so a contributor changing the stage's prompt can update the budget at the same time.

---

## Stage Patterns and Recommended Tiers

| Stage Type | Input Range | Output Range | Recommended Tier | Notes |
|------------|------------|--------------|------------------|-------|
| **Discover** | Large (research dumps, multiple docs) | Small (extracted findings) | Standard | Heavy input → cache aggressively |
| **Plan** | Medium (problem framing + constraints) | Large (structured plan) | Premium | Quality of plan compounds; spend here |
| **Build** | Medium (plan + relevant context only) | Large (artifacts) | Standard | Cache plan and constraints across iterations |
| **Refine / Critique** | Large (full artifact + spec) | Small (focused feedback) | Premium | Critique benefits from stronger reasoning; small output keeps cost in check |
| **Validate / Eval** | Small per case, many cases | Small per case | Cheap | Volume-driven - model tier is the dominant lever |
| **Operate / Monitor** | Tiny (alert / event payload) | Tiny (verdict / action) | Cheap | Latency matters more than quality |

---

## The Three Cost-Reduction Levers (per stage)

When a stage's cost is over-budget, attack the levers in this order - they correspond to the impact ranking in the table above.

### Lever 1 - Cut Context (highest impact, lowest risk)

The single most effective cut. Most over-context happens because the playbook author dumps "everything potentially relevant" instead of "what this stage actually needs."

| Tactic | When to Use |
|--------|-------------|
| **Per-stage context filtering** | The stage only needs sections X, Y of the upstream artifact, not the whole thing |
| **Extractive summarization upstream** | A prior stage produces a 200-token summary that downstream stages consume instead of the 4000-token raw output |
| **Reference, don't quote** | Embed a stable identifier and look up the content lazily instead of inlining |
| **Cache static context** | System prompts, tool definitions, evergreen reference material - cache once, reuse |

### Lever 2 - Match Model to Task

Most playbooks default to a Premium model everywhere. Most stages don't need it.

| Stage Type | If You're Using Premium, Try Dropping To |
|------------|------------------------------------------|
| Validate / Eval | Cheap (often 80% of the quality at 20% of the cost) |
| Operate / Monitor | Cheap (latency benefits too) |
| Build (after a strong Plan stage) | Standard (the plan does the heavy lifting) |
| Refine / Critique | **Stay on Premium** - critique is where reasoning quality compounds |

Always re-eval after a tier drop. The cost-quality-frontier skill in [AI-Eval-Skills](https://github.com/varunk130/AI-Eval-Skills) is the right tool for this.

### Lever 3 - Tame Reasoning Verbosity

Reasoning models often spend disproportionate output tokens on internal deliberation. For stages where reasoning quality is critical (Plan, Refine), keep verbosity high. For other stages:

- Constrain output schema (JSON with a maximum field count)
- Set explicit `max_tokens` limits per stage
- Provide examples that *demonstrate* terse output - the model imitates the example length

---

## Per-Playbook Budget Card

Every playbook in this library should ship a **Budget Card** at the top of its README, like:

```
## Budget Card
| Stage          | Input | Output | Tier     | Cost Notes                    |
|----------------|-------|--------|----------|-------------------------------|
| Discover       | ~8k   | ~1k    | Standard | Cache reference docs          |
| Plan           | ~3k   | ~4k    | Premium  | Quality compounds — spend     |
| Build          | ~5k   | ~6k    | Standard | Iterate; cache plan+context   |
| Refine         | ~10k  | ~1k    | Premium  | Critique stage                |
| Validate       | ~2k × N cases | ~0.5k × N | Cheap | Volume-driven                 |
```

The card lets a team:
- Estimate per-run cost for *their* model prices
- Identify the single most expensive stage at a glance
- Track regressions when a stage's expected ranges drift

---

## Sensitivity to Model Price Movement

Model prices move (often downward, occasionally upward when premium tiers launch). The Budget Card pattern is **price-agnostic** - it captures token usage and tier choice, not absolute dollars. To compute current cost:

```
estimated_cost_per_run = Σ_stages (input_tokens × in_price[tier] + output_tokens × out_price[tier])
```

Maintain a `pricing.json` (per the [cost-quality-frontier](https://github.com/varunk130/AI-Eval-Skills) pattern) with current model prices keyed by tier. When prices change, update the file once; every playbook's estimated cost updates downstream.

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| **Single token budget for the whole playbook** | Hides which stage is the expensive one; prevents targeted cuts |
| **"Use the best model everywhere"** | Single largest cost mistake; most stages don't need premium |
| **No per-stage caching strategy** | Iterative stages (Build, Refine) re-pay for static context every turn |
| **Verbosity-defaults on reasoning models** | 3-5× output cost regression with marginal quality gain |
| **Budget set once and never re-measured** | Drift is silent; instrument actual usage and compare to the card |

---

## Implementing in This Library

To adopt this guidance:

1. Add a `## Budget Card` section to each playbook's README (use the template above)
2. For each stage, fill in expected token ranges and recommended tier
3. List 1-3 cost-reduction levers per stage (use Lever 1/2/3 above as a starting menu)
4. Optionally: ship a `pricing.json` with the playbook so per-run cost estimates are computable

Stages without a budget card aren't broken - they just don't have a defensible answer to *"what does this cost per use?"* The card makes that answer explicit and improvable.
