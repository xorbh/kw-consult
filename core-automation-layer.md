# Core Automation Layer

## Overview

The Core Automation Layer is the infrastructure foundation that powers every client deployment. It is a pre-configured, hardened VM containing the essential components needed to ingest data, coordinate workflows, and enable AI agents to act on real systems.

Rather than asking clients to provision and maintain their own integration infrastructure — or cobbling together cloud services that require dedicated DevOps — we ship a ready-to-run environment that handles the plumbing so we can focus on the intelligence.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Client VM                          │
│                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ PostgreSQL   │  │    Redis    │  │   MinIO /    │ │
│  │              │  │             │  │ Object Store │ │
│  │ Primary data │  │ Caching,    │  │ File & doc   │ │
│  │ store, audit │  │ queues,     │  │ staging      │ │
│  │ logs, state  │  │ pub/sub     │  │              │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │                    n8n                           │ │
│  │                                                  │ │
│  │  Workflow orchestration, scheduling, webhooks,   │ │
│  │  API integrations, data transformations          │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Agent Runtime                       │ │
│  │                                                  │ │
│  │  Agent execution, tool access, LLM routing,     │ │
│  │  interaction tracing, skill invocation           │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
└─────────────────────────────────────────────────────┘
        │              │              │
        ▼              ▼              ▼
   Client ERP     Client CRM    Email / Chat
   & databases    & platforms    & messaging
```

## Core Components

### PostgreSQL

The primary data store for the automation layer.

- **Staging tables**: Ingested data from client systems lands here first — cleaned, normalised, and ready for agents and workflows to consume
- **State management**: Tracks the state of running workflows, agent sessions, and long-running processes
- **Audit & trace logs**: Every action taken by an agent or workflow is recorded with full context — who triggered it, what data was accessed, what the outcome was
- **Configuration store**: Agent definitions, skill mappings, connection credentials (encrypted), and governance policies

### Redis

In-memory data store for performance-critical operations.

- **Caching**: Frequently accessed data from client systems cached to reduce API load and latency
- **Message queues**: Asynchronous job processing — agents can queue tasks, workflows can hand off between stages
- **Pub/sub**: Real-time event distribution — when a workflow completes or an agent needs attention, subscribers are notified immediately
- **Rate limiting and session management**: Controls for API throttling, concurrent agent sessions, and resource allocation

### n8n

Visual workflow orchestration engine — the middleware coordination hub.

- **Data ingestion pipelines**: Scheduled and event-driven flows that pull data from client systems (APIs, databases, file drops, emails) into the staging layer
- **Transformation and enrichment**: Clean, reshape, and enrich data before it reaches agents or downstream systems
- **System integration**: Pre-built connectors for hundreds of platforms — ERP, CRM, accounting, HR, email, messaging, file storage
- **Scheduling and triggers**: Cron-based schedules, webhook listeners, database triggers, and manual invocations
- **Human-in-the-loop**: Approval steps, notification gates, and escalation paths built directly into workflows

### Object Storage

File and document staging.

- **Document ingestion**: Client documents, reports, spreadsheets, and attachments are stored and indexed for agent access
- **Export staging**: Generated reports, summaries, and outputs are staged here before delivery to client systems
- **Versioning**: Document versions tracked for audit and rollback

### Agent Runtime

The execution environment for AI agents, integrated directly into the automation layer.

- **Tool access**: Agents can query PostgreSQL, trigger n8n workflows, read from object storage, and interact with cached data in Redis — all through controlled interfaces
- **LLM routing**: Calls to language models are routed, logged, and traced through the management layer
- **Skill execution**: Agents invoke defined skills which map to specific tool chains, data sources, and permitted actions
- **Guardrails**: Agents operate within defined boundaries — approved data sources, action allowlists, spend limits on LLM calls, and escalation triggers

## What This Enables

**For agents:**
- Access to staged, clean data rather than raw client system APIs
- Ability to trigger and monitor workflows as part of their task execution
- Persistent memory and state across sessions via PostgreSQL
- Fast access to frequently needed data via Redis

**For workflows:**
- Reliable data pipelines that feed agents and downstream systems
- Middleware coordination between systems that were never designed to talk to each other
- Scheduled operations that run without human intervention but with full auditability

**For clients:**
- A single, managed environment rather than a sprawl of point-to-point integrations
- Data stays within their infrastructure boundary (on-prem or dedicated cloud VM)
- No need to build or maintain integration infrastructure themselves

## Deployment Model

Each client gets their own dedicated VM with the full stack pre-installed and configured. The VM can be deployed:

- **On-premises**: Within the client's own data centre or private cloud
- **Dedicated cloud**: A dedicated VM in our managed cloud infrastructure, isolated per tenant
- **Hybrid**: Core automation layer in the client's environment, with management layer connectivity back to KW Consult for monitoring and updates

The entire stack is containerised and configuration-managed, enabling:

- Reproducible deployments across clients
- Version-controlled infrastructure updates
- Rapid provisioning of new environments
- Consistent security baselines regardless of deployment location

## Relationship to Other Layers

The Core Automation Layer is the execution foundation. It works in concert with:

- **[AI Management Layer](ai-management-layer.md)**: Provides the administrative, tracing, and governance capabilities that sit above the automation layer
- **[AI Training Bootcamp](ai-training-bootcamp.md)**: Discovery insights from bootcamps inform what data pipelines, integrations, and agent capabilities get configured in the automation layer
- **[AI Services Model](ai-services-company.md)**: The automation layer is what makes the service model deliverable — it is the infrastructure that turns agent designs into running systems
