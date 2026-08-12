# Research: Simple Agent Chat App

**Date**: 2026-08-12 | **Feature**: `001-agent-chat-app`

## Decision 1: Backend framework and streaming transport

- **Decision**: FastAPI + Server-Sent Events (SSE) for agent reply streaming
- **Rationale**: FastAPI is lightweight for a local demo, has clear health-route ergonomics, and SSE is a simple browser-friendly stream (one request, progressive text events) without WebSocket complexity.
- **Alternatives considered**:
  - WebSockets — bidirectional overkill for v1 one-way reply streams
  - Full JSON response only — fails streaming acceptance criteria
  - Next.js Route Handlers only — blurs backend health endpoint and agent boundary for this split frontend/backend requirement

## Decision 2: Agent reply source for v1

- **Decision**: Pluggable stream producer with a **local stub agent** by default (deterministic chunked Traditional Chinese–capable text), optional OpenAI-compatible HTTP upstream via env if present
- **Rationale**: Spec defers model provider; stub keeps the demo runnable without API keys while still exercising true streaming UI behavior.
- **Alternatives considered**:
  - Hard-require cloud LLM — blocks local acceptance without secrets
  - Non-streaming mock — fails SC/FR streaming requirements

## Decision 3: Frontend stack and env configuration

- **Decision**: Vite + React + TypeScript; backend base URL from `VITE_BACKEND_URL` (documented in `.env.example`)
- **Rationale**: Vite exposes `VITE_*` env vars to the browser build/dev server, matching FR-007; React is sufficient for a single-thread chat UI.
- **Alternatives considered**:
  - Plain static HTML without build tooling — harder clean env injection story
  - Next.js — heavier than needed for v1 local demo

## Decision 4: Session/thread persistence

- **Decision**: Frontend in-memory thread only; backend may keep ephemeral request-scoped stream state only
- **Rationale**: Spec forbids database and allows refresh to clear history.
- **Alternatives considered**: LocalStorage persistence — out of stated v1 needs; adds scope

## Decision 5: Health/status endpoint shape

- **Decision**: `GET /health` returning JSON `{ "status": "ok" }` with HTTP 200 when process is up
- **Rationale**: Trivial independently checkable endpoint for FR-006 / SC-003
- **Alternatives considered**: Deep dependency checks — unnecessary without DB/external mandates

## Decision 6: Empty message and error handling

- **Decision**: Client-side reject empty/whitespace submits; on stream/network failure, mark in-progress agent message failed and show visible error, allow retry send
- **Rationale**: Matches edge cases and FR-009/FR-010 without backend complexity
- **Alternatives considered**: Silent failures — violates acceptance quality
