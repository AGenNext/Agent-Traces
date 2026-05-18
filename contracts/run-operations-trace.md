# Run Operations Trace Contract

## Purpose

Users need to inspect historical runs, scheduled runs, long-blocked runs, timed-out processes, exhausted tokens, expired API keys, failed tools, and agent ownership/blocker state.

This contract defines the trace and status model required for an operations dashboard.

## Core Views

Agent-Dashboard should support:

1. Past Run Logs
2. Scheduled Runs
3. Active Runs
4. Blocked Runs
5. Timed-Out Runs
6. Failed Runs
7. Token / Budget Exhaustion
8. Expired API Keys / Secret Failures
9. Tool Failure Dashboard
10. Agent Work Queue
11. Blocked By / Blocking Whom Map
12. SLA / Ageing Dashboard

## Run Statuses

```text
queued
scheduled
running
waiting_on_agent
waiting_on_human
waiting_on_tool
waiting_on_secret
waiting_on_budget
waiting_on_rate_limit
blocked
timeout_risk
timed_out
failed
completed
cancelled
approved
rejected
changes_requested
```

## Required Run Record Fields

```yaml
run_id: string
task_id: string
objective_id: string | null
conversation_id: string | null
workspace_id: string | null
tenant_id: string | null
status: string
priority: low | normal | high | urgent
owner_agent: string | null
current_agent: string | null
blocked_by_type: human | agent | tool | secret | budget | model | dependency | rate_limit | timeout | unknown | null
blocked_by_id: string | null
blocked_reason: string | null
blocked_since: datetime | null
blocked_duration_seconds: integer | null
scheduled_for: datetime | null
started_at: datetime | null
last_heartbeat_at: datetime | null
last_event_at: datetime | null
timeout_at: datetime | null
completed_at: datetime | null
sla_due_at: datetime | null
retry_count: integer
max_retries: integer
cost_so_far_usd: number | null
budget_limit_usd: number | null
tokens_used: integer | null
token_limit: integer | null
model_id: string | null
tool_ids: []
secret_ids: []
error_type: string | null
error_message: string | null
metadata: object
```

## Required Event Types

### Run lifecycle

- `run.scheduled`
- `run.queued`
- `run.started`
- `run.heartbeat`
- `run.status_changed`
- `run.completed`
- `run.failed`
- `run.cancelled`
- `run.timed_out`

### Blocking

- `run.blocked`
- `run.unblocked`
- `agent.blocked`
- `tool.blocked`
- `human.input.required`
- `secret.required`
- `budget.required`
- `rate_limit.waiting`

### Scheduling

- `schedule.created`
- `schedule.updated`
- `schedule.paused`
- `schedule.resumed`
- `schedule.triggered`
- `schedule.failed`

### Resource / quota / secrets

- `model.token_limit.reached`
- `model.context_limit.reached`
- `model.rate_limit.reached`
- `tool.api_key.expired`
- `tool.api_key.invalid`
- `tool.quota_exhausted`
- `secret.expired`
- `secret.rotation_required`
- `budget.exhausted`

## Blocked Run Classification

A blocked run must specify:

```yaml
blocked_by_type: human | agent | tool | secret | budget | model | dependency | rate_limit | timeout | unknown
blocked_by_id: string
blocked_reason: string
blocked_since: datetime
owner_agent: string
current_agent: string
```

Example:

```json
{
  "event_type": "run.blocked",
  "task_id": "rfp-123",
  "status": "blocked",
  "owner_agent": "project-manager-agent",
  "current_agent": "research-agent",
  "blocked_by_type": "tool",
  "blocked_by_id": "web-search-api",
  "blocked_reason": "API key expired",
  "blocked_since": "2026-05-18T10:30:00Z"
}
```

## Timeout Detection

A run is at timeout risk when:

```text
now > timeout_at - warning_window
```

A run is timed out when:

```text
now > timeout_at
AND status is not completed/cancelled/failed
```

Emit:

- `run.timeout_risk`
- `run.timed_out`

## Agent Heartbeats

Long-running agents must emit heartbeats:

```text
run.heartbeat
agent.heartbeat
```

If heartbeat is missing beyond threshold:

```text
agent.unresponsive
run.blocked
```

## Dashboard Queries

### Past run logs

```sql
SELECT * FROM runtime_events
WHERE event_type IN ['run.completed', 'run.failed', 'run.cancelled', 'run.timed_out']
ORDER BY created_at DESC;
```

### Runs blocked longer than threshold

```sql
SELECT * FROM runtime_events
WHERE event_type = 'run.blocked'
  AND blocked_since < $threshold
ORDER BY blocked_since ASC;
```

### Expired tool API keys

```sql
SELECT * FROM runtime_events
WHERE event_type IN ['tool.api_key.expired', 'tool.api_key.invalid', 'secret.expired']
ORDER BY created_at DESC;
```

### Token exhaustion

```sql
SELECT * FROM runtime_events
WHERE event_type IN ['model.token_limit.reached', 'model.context_limit.reached', 'budget.exhausted']
ORDER BY created_at DESC;
```

### Scheduled runs

```sql
SELECT * FROM runtime_events
WHERE event_type IN ['schedule.created', 'schedule.triggered', 'schedule.failed']
ORDER BY created_at DESC;
```

## Operations Dashboard Cards

Recommended cards:

- Total runs today
- Running now
- Scheduled next
- Blocked > 15 min
- Blocked > 1 hour
- Timed out
- Failed tools
- Expired secrets
- Token/context limit failures
- Budget exhaustion
- Waiting on human
- Waiting on agent

## Final Rule

Every run must have enough status, ownership, heartbeat, blocker, timeout, quota, and secret metadata for a user to answer:

```text
What ran?
What is running?
What is scheduled?
What is blocked?
Who/what is it blocked by?
How long has it been blocked?
What failed?
What needs human action?
```
