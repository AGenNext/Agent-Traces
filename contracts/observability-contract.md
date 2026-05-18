# Observability Contract

## Governing Principle

```text
Every important action should be traceable.
```

## Required Correlation Fields

- trace_id
- span_id
- parent_span_id
- objective_id
- task_id
- handoff_id
- agent_id
- model_run_id
- deployment_id
- environment
- tenant_id (when applicable)
- workspace_id (when applicable)

## Required Signals

- traces
- metrics
- structured logs
- events

## Standards

Prefer alignment with OpenTelemetry.

## Final Rule

If an outcome cannot be traced, it cannot be reliably debugged or trusted.
