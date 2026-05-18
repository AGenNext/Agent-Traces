# Agent Tool Usage Trace Contract

## Purpose

Agent-tool calls must be traceable as first-class runtime evidence.

This lets humans and agents answer:

- Which agent used which tool?
- Why was the tool called?
- What inputs were passed?
- What output was returned?
- Did the tool fail?
- How long did it take?
- What did it cost?
- Which loop, exchange, or artifact did it affect?

## Required Event Types

- `tool.call.started`
- `tool.call.completed`
- `tool.call.failed`
- `tool.call.skipped`
- `tool.call.retried`

## Required Fields

```yaml
tool_call_id: string
trace_id: string | null
span_id: string | null
parent_span_id: string | null
conversation_id: string | null
exchange_id: string | null
loop_id: string | null
iteration: integer | null
task_id: string
objective_id: string | null
agent_id: string
tool_id: string
tool_name: string
tool_provider: string | null
tool_version: string | null
event_type: tool.call.started | tool.call.completed | tool.call.failed | tool.call.skipped | tool.call.retried
status: started | completed | failed | skipped | retried
started_at: datetime | null
completed_at: datetime | null
latency_ms: integer | null
input_ref: string | null
input_summary: string | null
output_ref: string | null
output_summary: string | null
error_type: string | null
error_message: string | null
retry_count: integer
cost_usd: number | null
tokens_input: integer | null
tokens_output: integer | null
artifact_refs: []
source_refs: []
metadata: object
```

## Tool Call Lifecycle

```text
tool.call.started
  → tool.call.completed
```

or:

```text
tool.call.started
  → tool.call.failed
  → tool.call.retried
  → tool.call.completed
```

## Single-Agent Loop View

For a single agent loop, group by:

```text
task_id + agent_id + loop_id + iteration
```

Then show all tool calls under that iteration.

Example:

```text
Agent: research-agent
Loop: research-loop-1
Iteration: 2
  tool.call.started   arxiv_search
  tool.call.completed arxiv_search
  tool.call.started   web_fetch
  tool.call.completed web_fetch
```

## Multi-Agent Tool Usage View

For multi-agent workflows, group by:

```text
task_id → agent_id → loop_id → tool_call_id
```

This shows which agents used which tools and when.

## Dashboard Views

Agent-Dashboard should support:

1. Tool calls by objective
2. Tool calls by agent
3. Tool calls by loop/iteration
4. Failed tool calls
5. Tool latency table
6. Tool cost table
7. Tool input/output evidence links

## SurrealDB Query Examples

### Tool calls for one objective

```sql
SELECT * FROM runtime_events
WHERE task_id = $task_id
  AND event_type CONTAINS 'tool.call'
ORDER BY created_at ASC;
```

### Failed tool calls

```sql
SELECT * FROM runtime_events
WHERE task_id = $task_id
  AND event_type = 'tool.call.failed'
ORDER BY created_at ASC;
```

### Tool usage by agent

```sql
SELECT agent_id, tool_name, count() AS usage_count
FROM runtime_events
WHERE task_id = $task_id
  AND event_type = 'tool.call.completed'
GROUP BY agent_id, tool_name;
```

## Final Rule

Every external tool call must produce traceable started/completed or started/failed events with enough metadata to debug, audit, and evaluate the call.
