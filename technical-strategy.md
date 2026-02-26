# Technical Strategy: Anthropic-First

## The Decision

We are building exclusively on Anthropic as our base LLM provider. Every agent, every workflow with an LLM component, every piece of tooling we develop — all of it targets Claude models and the Anthropic API.

This is not a temporary convenience. It is a deliberate strategic choice.

## Why Single-Provider

The instinct in the market right now is to be "model-agnostic" — build abstraction layers, support multiple providers, let the client choose. This sounds reasonable but in practice it means:

- **Shallow integration everywhere**: You learn the lowest common denominator across providers rather than mastering the capabilities of any one. Every provider has different strengths in tool use, agentic behaviour, context handling, and instruction following. Abstracting over these differences means exploiting none of them.
- **Constant churn**: Every provider ships breaking changes, new features, and deprecations on different timelines. Supporting multiple providers means your engineering effort is spread across compatibility maintenance rather than capability development.
- **Diluted expertise**: Your team becomes generalists across five APIs instead of experts in one. When something goes wrong in production — and it will — depth of understanding is what gets you to a fix fast.
- **Testing surface explosion**: Every prompt, every tool chain, every agent behaviour needs to be validated across multiple models with different characteristics. The QA burden scales multiplicatively, not additively.

We reject this. Depth beats breadth.

## Why Anthropic

### Technical Fit

- **Best-in-class tool use**: Claude's tool use implementation is the most reliable and well-structured for agentic workflows. Agents that need to query databases, trigger APIs, read documents, and take multi-step actions work better on Claude than alternatives we have evaluated.
- **Long context handling**: Claude's extended context window and its ability to reason coherently over large inputs is critical for our use cases — ingesting business documents, analysing workflow traces, processing structured data from client systems.
- **Instruction following**: Our agents operate within strict guardrails — approved actions, data boundaries, compliance policies. Claude's ability to follow complex system prompts and respect constraints consistently is a baseline requirement, not a nice-to-have.
- **Agentic architecture support**: Anthropic's investment in agentic capabilities — including the Claude Code SDK, MCP (Model Context Protocol), and extended thinking — aligns directly with the kind of systems we build. We are building on a provider that is actively developing the primitives we need.

### Strategic Fit

- **API-first orientation**: Anthropic's business is the API. They are not competing with us to build the application layer, the workflow orchestration, or the enterprise admin interface. They provide the intelligence; we provide everything around it.
- **Enterprise trajectory without enterprise lock-in**: Anthropic is building enterprise features (SSO, audit logs, usage controls) but is not trying to own the full stack the way Microsoft (Copilot) or Google (Vertex AI) are. This leaves room for us to be the operational layer without being disintermediated.
- **Alignment and safety posture**: Our clients are in regulated industries and traditional enterprises where trust matters. Anthropic's public commitment to safety and responsible AI development is a selling point, not a constraint.

## What This Means in Practice

### For Our Engineering

- All prompt engineering, agent design patterns, and tool chain configurations are optimised for Claude models
- We build deep familiarity with Claude's behaviour — its strengths, edge cases, failure modes, and optimal prompting strategies
- We track Anthropic's model releases and API changes closely, adopting new capabilities early and integrating them into our stack before competitors
- We contribute to and build on Anthropic's ecosystem — MCP servers, Claude Code extensions, SDK tooling

### For Our Agents

- Agent prompts are written for Claude, not for a generic LLM
- Tool use schemas, response parsing, and error handling are all tuned to Claude's specific patterns
- We leverage Claude-specific features (extended thinking, multi-turn tool use, structured outputs) that are not available or not reliable on other providers
- Testing and quality assurance is focused and thorough rather than spread thin

### For Our Clients

- Consistent, predictable agent behaviour — one model family means fewer surprises
- Faster deployment — no time spent evaluating or switching between providers
- Clear cost model — Anthropic API pricing is the only variable, no need to optimise across multiple billing structures
- When Anthropic ships improvements, every client benefits immediately — we upgrade once, not per-provider

## Risk Acknowledgement

Single-provider dependency carries risk:

- **Pricing changes**: Anthropic could raise API prices. Mitigation: our value is in the orchestration and operational layer, not in reselling API calls. Even with price increases, the overall cost of our service remains a fraction of hiring equivalent headcount.
- **Service disruption**: Outages happen. Mitigation: the core automation layer (n8n, PostgreSQL, Redis) continues to operate during LLM outages. Agents degrade gracefully — queuing requests and resuming when service returns.
- **Capability stagnation**: Another provider could leap ahead. Mitigation: we monitor the market continuously. If a fundamental shift occurs, our agent architecture (skills, tool chains, workflows) is transferable — the provider-specific layer is the prompts and API integration, not the entire stack. But we do not optimise for a hypothetical switch that may never come.
- **Provider direction change**: Anthropic could move into our space. Mitigation: our value is domain-specific — understanding manufacturing workflows, procurement processes, financial operations. This is not a space Anthropic is likely to enter. They build general intelligence; we apply it.

## The Bet

We are betting that going deep on one provider — understanding it better than anyone, building tighter integrations, developing proprietary expertise in how Claude handles real operational workflows — creates more value than hedging across five providers and mastering none.

Specialisation is our competitive advantage. Anthropic is the platform we are specialising on.
