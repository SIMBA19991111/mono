# Implementation Plan: Simple Agent Chat App

**Branch**: `001-agent-chat-app` | **Date**: 2026-08-12 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-agent-chat-app/spec.md`

## Summary

Build a minimal local web chat app with a single in-session thread: users send Traditional Chinese messages from a browser UI and receive streamed text replies from a backend agent. v1 also exposes a backend health/status endpoint and configures the frontend’s backend base URL via environment variable. No login, database, RAG, tools, attachments, or production deployment.

## Technical Context

**Language/Version**: Python 3.12 (backend), TypeScript + Vite (frontend)

**Primary Dependencies**: FastAPI + Uvicorn (HTTP + SSE streaming); Vite + React (chat UI); httpx optional for upstream LLM if configured; otherwise a local deterministic streaming stub agent

**Storage**: N/A (in-memory session state only; no durable persistence)

**Testing**: Manual quickstart validation for v1; optional lightweight API checks with curl/httpx (no mandated automated test suite in spec)

**Target Platform**: Local developer machine (Linux/macOS/Windows) via browser + local processes

**Project Type**: Web application (frontend + backend)

**Performance Goals**: First streamed agent text visible within ~5 seconds under normal local conditions; stream updates without full-page refresh

**Constraints**: Single chat thread; no auth/DB/RAG/tools/uploads/prod deploy; Traditional Chinese round-trip must not corrupt characters; frontend backend URL from env var

**Scale/Scope**: Single local user, one browser session, minimal chat UI (message list + input + send)

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

`.specify/memory/constitution.md` is still the Spec Kit placeholder (principles not ratified). For this feature:

- **Simplicity**: PASS — two lightweight apps (backend + frontend), no extra services
- **No unjustified persistence/auth**: PASS — explicitly out of scope
- **Spec alignment**: PASS — plan maps 1:1 to FR/SC in spec.md
- **Placeholder constitution**: No enforceable MUST gates beyond keeping v1 minimal; no complexity tracking required

Post-design re-check: unchanged — design remains a simple SSE chat + health endpoint + env-configured frontend base URL.

## Project Structure

### Documentation (this feature)

```text
specs/001-agent-chat-app/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── health.openapi.yaml
│   └── chat-stream.openapi.yaml
└── tasks.md
```

### Source Code (repository root)

```text
backend/
├── pyproject.toml
├── .env.example
├── src/
│   ├── main.py
│   ├── api/
│   │   ├── health.py
│   │   └── chat.py
│   ├── agent/
│   │   └── stream.py
│   └── models/
│       └── chat.py
└── README.md

frontend/
├── package.json
├── .env.example
├── vite.config.ts
├── index.html
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── components/
│   │   ├── ChatThread.tsx
│   │   ├── MessageBubble.tsx
│   │   └── MessageInput.tsx
│   ├── config.ts
│   └── api/
│       └── chatClient.ts
└── README.md
```

**Structure Decision**: Web application layout with separate `backend/` and `frontend/` directories so the health endpoint, streaming chat API, and env-configured UI base URL stay clear and independently runnable for local demo use.

## Complexity Tracking

> No constitution violations requiring justification.
