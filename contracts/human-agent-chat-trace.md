# Human-Agent Chat Trace Contract

## Purpose

Human-agent conversations must be traceable as first-class workflow evidence.

This is required when a human repeatedly asks an agent for changes, clarification, rejection, approval, or revision.

## Core Idea

```text
conversation_id
  → exchange_id
  → message_id
  → revision_id
  → linked runtime/event/artifact changes
```

## Required Event Types

- `chat.conversation.started`
- `chat.message.sent`
- `chat.message.received`
- `chat.change_requested`
- `chat.agent_revision_started`
- `chat.agent_revision_completed`
- `chat.human_approved`
- `chat.human_rejected`
- `chat.conversation.closed`

## Required Fields

```yaml
conversation_id: string
exchange_id: string
message_id: string
parent_message_id: string | null
task_id: string
artifact_id: string | null
agent_id: string | null
human_id: string | null
role: human | agent | system
direction: human_to_agent | agent_to_human | system
message_type: request | response | change_request | approval | rejection | clarification | revision
revision_id: string | null
change_request_id: string | null
created_at: datetime
content_summary: string
content_ref: string | null
metadata: object
```

## Human-Agent Change Loop

Example:

```text
exchange-1: human → agent: create sales deck
exchange-1: agent → human: draft v1
exchange-2: human → agent: change pricing slide
exchange-2: agent → human: draft v2
exchange-3: human → agent: make it shorter
exchange-3: agent → human: draft v3
exchange-4: human → agent: approved
```

## Artifact Version Linkage

Every change request should link to the artifact version it changes.

```text
artifact:v1
  → change_request:cr-1
  → artifact:v2
  → change_request:cr-2
  → artifact:v3
  → approval
```

## Dashboard Views

The dashboard should support:

1. Conversation timeline
2. Change request history
3. Artifact version diff history
4. Human approval trail
5. Agent response trail

## Final Rule

If a human asks for changes multiple times, every turn must be preserved as an ordered, queryable conversation trace.
