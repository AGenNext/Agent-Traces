# Streaming Stage Trace Contract

## Purpose

Agent applications should show live progress as structured streaming stages.

This gives users visibility similar to LangChain/LangGraph-style agent streams without exposing hidden chain-of-thought.

## Core Principle

```text
Stream stages, events, tools, evidence, and decisions.
Do not stream private chain-of-thought.
```

## Common Streaming Stages

- `queued`
- `runtime.started`
- `planning.started`
- `planning.completed`
- `retrieval.started`
- `retrieval.completed`
- `tool.call.started`
- `tool.call.completed`
- `tool.call.failed`
- `agent.handoff.started`
- `agent.handoff.completed`
- `generation.started`
- `generation.delta`
- `generation.completed`
- `evaluation.started`
- `evaluation.completed`
- `trust.started`
- `trust.completed`
- `finops.started`
- `finops.completed`
- `human.review.required`
- `runtime.completed`
- `runtime.failed`

## Required Fields

```yaml
stage_event_id: string
trace_id: string | null
span_id: string | null
parent_span_id: string | null
task_id: string
objective_id: string | null
conversation_id: string | null
exchange_id: string | null
loop_id: string | null
iteration: integer | null
agent_id: string | null
stage: string
status: queued | running | completed | failed | skipped
message: string
progress_percent: number | null
started_at: datetime | null
completed_at: datetime | null
latency_ms: integer | null
tool_call_id: string | null
handoff_id: string | null
artifact_id: string | null
evidence_refs: []
metadata: object
created_at: datetime
```

## Example Stream

```text
Queued objective
Planning execution
Searching sources
Calling arxiv_search
Received 8 papers
Drafting artifact
Evaluating result
Checking provenance
Estimating cost
Ready for human review
```

## UI Pattern

A dashboard should render streaming stages as:

```text
✓ Runtime started
✓ Planning completed
✓ Retrieval completed
✓ Tool call: arxiv_search completed
✓ Generation completed
✓ Evaluation passed
✓ Trust conditional
✓ FinOps within budget
→ Human review required
```

## Streaming Transport Options

Recommended transports:

1. Server-Sent Events for simple one-way progress streams
2. WebSockets for bidirectional live collaboration
3. Polling `GET /objectives/{task_id}/events` for simplest MVP

## MVP Recommendation

Start with polling persisted runtime events every 1-2 seconds.

Later upgrade to Server-Sent Events:

```http
GET /objectives/{task_id}/stream
```

## Relationship to LangChain / LangGraph

LangChain and LangGraph-style apps usually expose live progress through callbacks, event streams, or graph node updates.

Agent Platform should normalize those into this stage contract so the dashboard is framework-independent.

## Final Rule

Users should always see what stage the agent is in, which tools are running, what completed, what failed, and what needs human attention — without exposing raw hidden reasoning.
