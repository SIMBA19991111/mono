# Quickstart: Simple Agent Chat App

**Feature**: `001-agent-chat-app` | **Date**: 2026-08-12

Validate the three acceptance criteria locally: streamed chat UI, health endpoint, and frontend backend URL via env.

## Prerequisites

- Python 3.12+
- Node.js 20+
- Two terminals

## 1. Start backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e .
cp .env.example .env
uvicorn src.main:app --reload --host 127.0.0.1 --port 8000
```

### Health check (AC2)

```bash
curl -s http://127.0.0.1:8000/health
```

Expected: `{"status":"ok"}` with HTTP 200. See [contracts/health.openapi.yaml](./contracts/health.openapi.yaml).

## 2. Start frontend with configurable backend URL (AC3)

```bash
cd frontend
cp .env.example .env
# Ensure VITE_BACKEND_URL=http://127.0.0.1:8000
npm install
npm run dev
```

Open the printed local URL (typically `http://127.0.0.1:5173`).

To prove env configurability: stop frontend, change `VITE_BACKEND_URL` to another reachable backend base URL, restart `npm run dev`, and confirm chat requests target the new host.

## 3. Streamed chat (AC1)

1. Type a Traditional Chinese message (e.g. `你好，請介紹你自己`) and send.
2. Confirm the user message appears in the single thread.
3. Confirm agent text appears progressively (streaming), then completes.
4. Send two more turns; all remain in the same thread.

Stream contract: [contracts/chat-stream.openapi.yaml](./contracts/chat-stream.openapi.yaml). Entities: [data-model.md](./data-model.md).

## Failure checks

- Empty submit: blocked in UI; no empty exchange.
- Stop backend mid-session: UI shows visible error within ~10s and allows retry after backend restart.
