# Agent Reasoning Summary Trace Contract

## Purpose

Agent systems should not expose hidden model chain-of-thought directly.

Instead, systems should expose an enterprise-safe reasoning summary trace that gives humans enough visibility to inspect, audit, debug, and approve agent behavior without leaking private scratchpad reasoning.

## Core Principle

```text
Do not expose private chain-of-thought.
Expose traceable reasoning summaries, decisions, evidence, tool usage, and outcomes.
```

## Why Not Raw Chain-of-Thought

Raw chain-of-thought may contain:

- private model reasoning
- unstable intermediate thoughts
- irrelevant speculation
- sensitive prompt/security details
- unverified assumptions
- policy-sensitive reasoning internals

Enterprise users need accountability and auditability, not raw hidden scratchpad text.

## Required Event Types

- `reasoning.summary.started`
- `reasoning.summary.completed`
- `decision.made`
- `assumption.recorded`
- `alternative.considered`
- `evidence.considered`
- `confidence.assessed`
- `next_action.selected`

## Required Fields

```yaml
reasoning_summary_id: string
trace_id: string | null
span_id: string | null
parent_span_id: string | null
task_id: string
objective_id: string | null
conversation_id: string | null
exchange_id: string | null
loop_id: string | null
iteration: integer | null
agent_id: string
event_type: string
summary: string
decision: string | null
rationale_summary: string | null
inputs_considered: []
evidence_refs: []
tool_call_refs: []
source_refs: []
alternatives_considered: []
assumptions: []
risks: []
confidence: number | null
next_action: string | null
created_at: datetime
metadata: object
```

## What Humans Should See

A dashboard should show:

1. What the agent was trying to do
2. What inputs it considered
3. Which tools it used
4. Which evidence it relied on
5. What alternatives it considered
6. What decision it made
7. Why, in concise summary form
8. What risks or assumptions remain
9. Confidence score
10. Next action

## Example

```json
{
  "event_type": "decision.made",
  "agent_id": "product-manager-agent",
  "loop_id": "pm-loop-1",
  "iteration": 2,
  "decision": "Request UX review before release",
  "rationale_summary": "The feature satisfies the functional requirement, but the user flow has not been validated against the dashboard usability checklist.",
  "evidence_refs": ["eval:ux-checklist-v1"],
  "tool_call_refs": ["tool-call:dashboard-check"],
  "alternatives_considered": ["ship immediately", "request code review only"],
  "risks": ["user approval flow may be confusing"],
  "confidence": 0.78,
  "next_action": "handoff to UX designer agent"
}
```

## Relationship to Other Trace Types

```text
Reasoning Summary Trace
  → explains decisions safely

Tool Usage Trace
  → shows tool calls and results

Human-Agent Chat Trace
  → shows human change requests and approvals

Runtime Event Trace
  → shows execution order

Trust Trace
  → links claims to evidence
```

## Dashboard Views

Agent-Dashboard should support:

- Agent decision timeline
- Reasoning summary per loop
- Reasoning summary per iteration
- Evidence used per decision
- Tool calls used per decision
- Assumptions and risks panel
- Confidence trend across iterations

## Final Rule

Never display hidden chain-of-thought as the audit record.

Display concise reasoning summaries grounded in evidence, decisions, tool calls, and outcomes.
