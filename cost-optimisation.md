# Cost Optimisation

## The Problem

When AI agents move from pilot to production, token consumption becomes a real operational cost. Left unmanaged, it grows unpredictably — verbose prompts, redundant context loading, unnecessary round-trips, and agents that brute-force tasks instead of working efficiently.

Most companies won't notice this during a proof of concept. They notice it when the monthly API bill arrives and it's three times what they expected. This is where adoption stalls and CFOs start asking hard questions.

Our job is to make sure that doesn't happen.

## Why This Is Our Problem to Solve

Because we own the full stack — the agents, the automation layer, the management layer, and the trace data — we are uniquely positioned to see where tokens are being spent and why. We don't just report on costs; we can actually fix them.

This is a core part of the managed service. Clients should not need to understand token economics. They should see a predictable cost that delivers measurable value.

## How We Optimise

### 1. Trace-Based Spend Analysis

Every LLM call passes through our management layer and is fully traced. This gives us:

- **Per-agent cost attribution**: Exactly how much each agent costs to run, per task, per day, per user
- **Per-skill cost breakdown**: Which skills (tool chains, workflows) are expensive and which are cheap
- **Token composition analysis**: How much of the token spend is system prompt, context, user input, tool results, and model output — broken down per call
- **Outlier detection**: Calls that consume significantly more tokens than expected for their task type — often the first sign of a poorly designed prompt or an unnecessary loop

### 2. Prompt Engineering for Efficiency

The single biggest lever for cost reduction is prompt design.

- **Context pruning**: Agents often load more context than they need. We analyse which parts of the system prompt and injected context are actually used in the model's reasoning and strip out the rest.
- **Dynamic context loading**: Instead of stuffing everything into every call, agents load context on demand — pulling in relevant data via tool calls only when needed.
- **Prompt compression**: Rewriting verbose instructions to be concise without losing effectiveness. Claude responds well to tightly written prompts; verbosity does not improve output quality.
- **Response shaping**: Instructing agents to return structured, concise outputs rather than verbose explanations — especially for intermediate steps that feed into downstream workflows rather than human consumption.

### 3. Architectural Optimisation

Some cost problems aren't about prompts — they're about how the agent is designed.

- **Task decomposition**: Breaking complex tasks into smaller, focused steps where each step uses a cheaper or faster model tier (e.g. Haiku for classification and extraction, Sonnet for reasoning, Opus for complex judgement calls).
- **Caching layers**: Using Redis to cache responses for repeated or similar queries. If ten users ask the same question about the same dataset within an hour, we don't need ten LLM calls.
- **Pre-computation**: Moving deterministic operations out of the LLM entirely. If a task can be solved with a SQL query, an n8n workflow, or a simple function, it should never hit the model.
- **Batch processing**: Grouping multiple small requests into single, structured calls where the model handles them together rather than one at a time.
- **Tool result compression**: When tool calls return large payloads (database query results, document contents), summarising or filtering them before injecting into the conversation context.

### 4. Model Tier Routing

Not every task needs the most capable model.

- **Haiku for simple tasks**: Classification, extraction, formatting, routing decisions, and validation checks. Fast and cheap.
- **Sonnet for standard tasks**: Most day-to-day agent work — answering questions, generating reports, processing workflows. Good balance of capability and cost.
- **Opus for complex tasks**: Multi-step reasoning, nuanced judgement, complex document analysis, and tasks where accuracy is critical and cost is secondary.

We implement intelligent routing that assesses the complexity of each request and directs it to the appropriate model tier. This is invisible to the end user — the agent works the same way, but the backend optimises for cost.

### 5. Continuous Monitoring and Feedback Loops

Optimisation is not a one-time exercise.

- **Cost dashboards**: Per-agent, per-skill, per-department cost tracking visible to both KW Consult and the client
- **Budget alerts**: Configurable thresholds that trigger notifications when spend exceeds expected levels
- **Weekly spend reports**: Automated summaries of where tokens went, what changed, and what we recommend
- **Regression detection**: When a prompt change or agent update causes a cost spike, we catch it immediately through trace comparison

## The Commercial Angle

Cost optimisation is not a separate service — it is embedded in how we operate. But it has clear commercial value:

- **Justifies the managed service fee**: Clients pay us to run their agents. Part of that value is that we actively manage costs downward. A client running agents themselves would overspend significantly.
- **Builds trust with finance teams**: When we can show a CFO exactly what they're spending, why, and how we're reducing it quarter over quarter — that's a relationship that renews.
- **Enables scale**: Unoptimised agents don't scale. A procurement agent that costs £5 per interaction is viable for ten users; it's not viable for five hundred. Optimisation is what makes organisation-wide rollout economically feasible.
- **Differentiates from DIY**: Any company can spin up Claude and start building. Few can operate it efficiently at scale. That's where we live.
