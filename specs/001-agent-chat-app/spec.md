# Feature Specification: Simple Agent Chat App

**Feature Branch**: `001-agent-chat-app`

**Created**: 2026-08-12

**Status**: Draft

**Input**: User description: "建立一個簡單的 agent chat app。使用者可在 web 介面輸入繁體中文訊息，並收到由 backend agent 串流回傳的回覆。v1 只需要單一聊天 thread；不包含登入、資料庫、RAG、tools、上傳附件或 production deployment。Acceptance criteria：1. web 介面可送出訊息並顯示串流回覆。2. backend 有可檢查的 health/status endpoint。3. 前端 endpoint 可透過 environment variable 設定。"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Send a message and see a streamed reply (Priority: P1)

A visitor opens the chat web page, types a Traditional Chinese message into the input box, and sends it. The page shows their message in the single chat thread, then displays the agent reply as it arrives in a streaming fashion (text appears progressively rather than only after the full response is ready).

**Why this priority**: This is the core value of the product. Without send + streamed reply in one thread, there is no usable chat experience.

**Independent Test**: Open the web UI, send one Traditional Chinese message, and confirm the user message appears and the agent reply streams into the same thread until complete.

**Acceptance Scenarios**:

1. **Given** the chat page is open with an empty thread, **When** the user submits a Traditional Chinese message, **Then** the message appears in the thread and a streamed agent reply begins rendering in the same thread.
2. **Given** the agent is actively streaming a reply, **When** new text chunks arrive, **Then** the UI updates the in-progress reply without requiring a page refresh.
3. **Given** a completed exchange exists in the thread, **When** the user sends another message, **Then** the new exchange is appended to the same single thread (no thread switching or creation).

---

### User Story 2 - Verify backend health (Priority: P2)

An operator or developer needs a simple way to confirm the backend is running and reachable before or while using the chat UI, by calling a health/status endpoint and receiving a clear healthy response.

**Why this priority**: Required for basic operability and for the stated acceptance criteria, but secondary to the chat experience itself.

**Independent Test**: Call the backend health/status endpoint and confirm it returns a successful healthy status without using the chat UI.

**Acceptance Scenarios**:

1. **Given** the backend is running normally, **When** a client requests the health/status endpoint, **Then** the response indicates the service is healthy/available.
2. **Given** the backend is not running, **When** a client requests the health/status endpoint, **Then** the request fails in a way that makes unavailability obvious (no false healthy response).

---

### User Story 3 - Point the web UI at a configurable backend (Priority: P2)

A developer configures where the web UI should send chat requests by setting an environment variable, then starts the UI and successfully chats against that backend location without hardcoding the address in source that cannot be changed per environment.

**Why this priority**: Needed for flexible local development and for the stated acceptance criteria; not needed for a single hardcoded demo, but required for v1 acceptance.

**Independent Test**: Set the frontend backend-endpoint environment variable to a known backend address, start the web UI, send a message, and confirm the request goes to that configured endpoint.

**Acceptance Scenarios**:

1. **Given** a backend endpoint is provided via environment variable, **When** the web UI is started, **Then** chat requests are directed to that configured endpoint.
2. **Given** the environment variable is changed to a different valid backend endpoint and the UI is restarted (or reconfigured as documented), **When** the user sends a message, **Then** the UI uses the new endpoint.

---

### Edge Cases

- What happens when the user submits an empty or whitespace-only message? The system rejects or ignores the submit and does not create an empty exchange.
- What happens when the backend is unreachable or returns an error during send/stream? The UI shows a clear error state and does not leave the thread stuck in a permanent “streaming” state.
- What happens if the stream ends mid-reply (connection drop)? The UI stops streaming, marks the reply as incomplete/failed with a user-visible message, and allows the user to send another message.
- How does the system handle very long Traditional Chinese input? The message is accepted within a reasonable practical length; if over limit, the UI prevents send or shows a clear validation message.
- What happens on page refresh? v1 keeps only a single in-session thread and does not promise durable history; refresh may clear the visible conversation.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide a web chat interface where a user can compose and send Traditional Chinese text messages.
- **FR-002**: System MUST maintain exactly one chat thread in v1 (no create/list/switch thread flows).
- **FR-003**: System MUST display the user’s sent messages in the chat thread in chronological order.
- **FR-004**: System MUST obtain agent replies from a backend agent service and render them as a stream of text updates in the same thread.
- **FR-005**: System MUST support round-trip Traditional Chinese text for both user input and agent output display without mojibake or forced conversion to another language.
- **FR-006**: Backend MUST expose a health/status endpoint that callers can use to verify the service is available.
- **FR-007**: Frontend MUST obtain the backend endpoint address from an environment variable (not a permanently hardcoded unreachable value).
- **FR-008**: System MUST allow multiple sequential message/reply turns within the single thread during one browsing session.
- **FR-009**: System MUST present a clear error to the user when sending a message or receiving a stream fails.
- **FR-010**: System MUST prevent empty/whitespace-only messages from being sent as chat turns.

### Key Entities

- **Chat Thread**: The single conversation container for v1; holds an ordered sequence of messages for the current session.
- **Message**: A user or agent utterance with role (user/agent), text content, and completion state (pending, streaming, complete, failed).
- **Agent Reply Stream**: A progressive delivery of agent text for one user message until completion or failure.
- **Backend Endpoint Configuration**: The runtime setting that tells the web UI where to send chat and related requests.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can open the web UI, send a Traditional Chinese message, and see a streamed agent reply appear in the same thread within 5 seconds of first visible streamed text under normal local conditions.
- **SC-002**: 100% of successful chat sends show both the user message and a completing agent reply in the single thread without requiring a manual page refresh.
- **SC-003**: The backend health/status endpoint returns a successful healthy indication when the service is up, and can be checked independently of the chat UI.
- **SC-004**: Changing the frontend backend-endpoint environment variable and restarting/reloading as documented causes subsequent chat requests to target the new endpoint on the next successful send.
- **SC-005**: Users can complete at least 3 sequential message/reply turns in one session without leaving the page or creating a new thread.
- **SC-006**: When the backend is stopped, the UI surfaces a visible failure for send/stream within 10 seconds and remains usable to retry after the backend recovers.

## Assumptions

- v1 is a local/demo experience for a single user in one browser session; concurrency across many users is out of scope.
- No authentication, authorization, or multi-user isolation is required.
- No persistent database or durable chat history is required; in-memory/session display is acceptable.
- No RAG, tool calling, file/image attachments, or production deployment tooling are included.
- The “backend agent” can produce a textual streamed reply suitable for chat display; exact model provider is an implementation concern deferred to planning.
- “Environment variable” configuration applies to how the frontend is started/built for local use; operators will set it before use.
- Desktop web browser use is the primary target; dedicated mobile app packaging is out of scope.
- A minimal visual chat layout (message list + input + send) is sufficient; advanced UX polish is out of scope for v1.
