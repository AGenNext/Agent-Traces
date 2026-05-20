# Agent-Traces boundary

Agent-Traces owns trace, audit, execution timeline, and observability contracts for AGenNext.

## Scope

Agent-Traces owns:

- execution traces
- decision traces
- tool/action traces
- audit event contracts
- workflow timelines
- runtime event correlation
- incident timelines
- trace query models
- trace export contracts

Agent-Traces does not own:

- memory storage primitives
- GraphRAG retrieval
- runtime execution
- identity verification
- Kubernetes operations

## Boundary

| Component | Responsibility |
|---|---|
| Agent-Traces | Trace/audit/timeline contracts |
| Agent-Memory | Durable memory storage primitives |
| Agent-RAG | Retrieval and GraphRAG pipelines |
| Agent-Runtime | Executes workflows and emits trace events |
| Agent-Dashboard | Displays traces and audit timelines |
| SurrealDB | Default persistence backend for trace records |

## Flow

```txt
Agent-Runtime executes node
  ↓
emits trace event
  ↓
Agent-Traces contract validates event
  ↓
event is persisted to SurrealDB or exported
  ↓
Agent-Dashboard displays timeline
```

## Rule

Trace and audit models should not be hidden inside Agent-Memory.

Agent-Memory stores memory.

Agent-Traces models execution history.
