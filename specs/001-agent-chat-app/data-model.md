# Data Model: Simple Agent Chat App

**Date**: 2026-08-12 | **Feature**: `001-agent-chat-app`

In-memory / client-side entities only. No durable storage.

## Entities

### ChatThread

- **Purpose**: Single v1 conversation container for the browser session
- **Fields**:
  - `id` (string, constant for session, e.g. `default`)
  - `messages` (ordered list of `Message`)
- **Rules**: Exactly one thread; no create/list/switch APIs

### Message

- **Purpose**: One user or agent utterance in the thread
- **Fields**:
  - `id` (string, unique in session)
  - `role` (`user` | `agent`)
  - `content` (string, UTF-8; may grow while streaming)
  - `status` (`pending` | `streaming` | `complete` | `failed`)
  - `createdAt` (ISO timestamp string)
  - `error` (optional string when `failed`)
- **Validation**:
  - User `content` MUST be non-empty after trim before send
  - Practical max length: 4000 characters (client reject with message if exceeded)
- **Transitions**:
  - User message: `pending` → `complete` (on accepted send) or `failed` (on send error before stream starts)
  - Agent message: `pending` → `streaming` → `complete` | `failed`

### AgentReplyStream

- **Purpose**: Progressive delivery of one agent reply
- **Fields**:
  - `messageId` (agent `Message.id`)
  - `chunks` (sequence of text fragments)
  - `done` (boolean)
  - `error` (optional string)
- **Rules**: Mapped 1:1 to a single agent `Message`; UI appends chunks to `content` until `done` or `error`

### BackendEndpointConfiguration

- **Purpose**: Tell the frontend where the backend lives
- **Fields**:
  - `baseUrl` (string, from `VITE_BACKEND_URL`, e.g. `http://127.0.0.1:8000`)
- **Rules**: Required for chat client; no permanently hardcoded unreachable production URL in source of truth

## Relationships

```text
ChatThread 1 ── * Message
Message(agent) 1 ── 0..1 AgentReplyStream
Frontend ── uses ── BackendEndpointConfiguration
```
