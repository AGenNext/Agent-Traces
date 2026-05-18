# Agent Traces

Agent Traces owns the canonical semantic trace, telemetry, and log contracts for AGenNext agentic systems.

It is the source of truth for how agent runs, tool calls, model calls, human-agent chat, loop iterations, blocked runs, schedules, approvals, and runtime operations are represented.

## Responsibility

Agent Traces defines how runtime behavior is observed and correlated across:

- objectives
- runs
- schedules
- blockers
- A2A handoffs
- agent execution loops
- human-agent chat
- tool calls
- model calls
- runtime events
- API requests
- background workers
- evaluations
- trust checks
- FinOps events
- deployments
- post-production incidents

## Scope

Agent Traces covers:

- trace IDs
- correlation IDs
- span conventions
- structured logs
- telemetry events
- runtime diagnostics
- error traces
- audit trace links
- OpenTelemetry alignment
- Langfuse adapter mapping
- SigNoz/OpenTelemetry adapter mapping
- dashboard observability contracts

## Source of Truth

```text
Agent-Traces
  → owns semantic observability contracts

Langfuse
  → optional/upstream adapter for LLM and agent observability

SigNoz
  → optional/upstream adapter for APM, metrics, logs, and infra traces

Agent-Dashboard
  → product/control-plane view over Agent-Traces-shaped data
```

## Consumers

- Agent-Frameworks
- Agent-Team
- Agent-Knowledge
- Agent-Platform
- Agent-deploy
- Agent-Analytics
- Agent-Dashboard
- Agent-Trust
- Agent-FinOps
- Model-Router

## Core Principle

```text
Every important action should be traceable across agents, services, tools, humans, artifacts, and time.
```

## Relationship to Other Repos

```text
Agent-Traces
  → observability contracts

Agent-Analytics
  → aggregates metrics and trends

Agent-deploy
  → deploys monitoring and alerting

Agent-Dashboard
  → visualizes traces and runtime status

Agent-Trust
  → links trace evidence to trust records

Agent-FinOps
  → links trace events to costs and budgets
```

## Final Rule

Own the semantic trace model in Agent-Traces.
Use Langfuse and SigNoz through adapters, not as the canonical product data model.
