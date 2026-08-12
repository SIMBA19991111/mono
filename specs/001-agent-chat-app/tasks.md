# Tasks: Simple Agent Chat App

**Input**: Design documents from `/specs/001-agent-chat-app/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Not requested in the feature specification — no automated test tasks included.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Web app: `backend/src/`, `frontend/src/`

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create `backend/` and `frontend/` directory trees per plan.md (including `backend/src/api`, `backend/src/agent`, `backend/src/models`, `frontend/src/components`, `frontend/src/api`)
- [ ] T002 Initialize Python backend package with FastAPI/Uvicorn dependencies in `backend/pyproject.toml`
- [ ] T003 [P] Initialize Vite + React + TypeScript app manifests in `frontend/package.json` and `frontend/vite.config.ts`
- [ ] T004 [P] Add backend env sample in `backend/.env.example`
- [ ] T005 [P] Add frontend env sample documenting `VITE_BACKEND_URL` in `frontend/.env.example`
- [ ] T006 [P] Write short run notes in `backend/README.md` and `frontend/README.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T007 Create FastAPI app entrypoint with CORS for local frontend origin in `backend/src/main.py`
- [ ] T008 [P] Define chat message/thread Pydantic models in `backend/src/models/chat.py`
- [ ] T009 [P] Create frontend config reader for `VITE_BACKEND_URL` in `frontend/src/config.ts`
- [ ] T010 Scaffold React root shell mounting a single chat page in `frontend/src/main.tsx` and `frontend/src/App.tsx`
- [ ] T011 [P] Add `frontend/index.html` Vite entry HTML

**Checkpoint**: Foundation ready — user story implementation can now begin

---

## Phase 3: User Story 1 - Send a message and see a streamed reply (Priority: P1) 🎯 MVP

**Goal**: User sends Traditional Chinese text and sees a streamed agent reply in one in-memory thread

**Independent Test**: Open UI, send one Traditional Chinese message, confirm user message appears and agent reply streams to completion in the same thread

### Implementation for User Story 1

- [ ] T012 [P] [US1] Implement stub (and optional upstream) streaming agent producer in `backend/src/agent/stream.py`
- [ ] T013 [US1] Implement `POST /api/chat/stream` SSE endpoint validating non-empty ≤4000 char messages in `backend/src/api/chat.py`
- [ ] T014 [US1] Register chat router in `backend/src/main.py`
- [ ] T015 [P] [US1] Implement SSE chat client using configured base URL in `frontend/src/api/chatClient.ts`
- [ ] T016 [P] [US1] Build message list UI in `frontend/src/components/ChatThread.tsx` and `frontend/src/components/MessageBubble.tsx`
- [ ] T017 [US1] Build input + send with empty/whitespace rejection in `frontend/src/components/MessageInput.tsx`
- [ ] T018 [US1] Wire App state for single thread, streaming append, failure/retry UX in `frontend/src/App.tsx`

**Checkpoint**: User Story 1 fully functional and independently testable (MVP)

---

## Phase 4: User Story 2 - Verify backend health (Priority: P2)

**Goal**: Independently checkable backend health/status endpoint

**Independent Test**: `curl` the health endpoint while backend is up and confirm healthy JSON; confirm failure when backend is down

### Implementation for User Story 2

- [ ] T019 [P] [US2] Implement `GET /health` returning `{"status":"ok"}` in `backend/src/api/health.py`
- [ ] T020 [US2] Register health router in `backend/src/main.py`

**Checkpoint**: Health check works without using the chat UI

---

## Phase 5: User Story 3 - Point the web UI at a configurable backend (Priority: P2)

**Goal**: Frontend targets backend base URL from environment variable

**Independent Test**: Set `VITE_BACKEND_URL`, start frontend, send a message; change URL and restart; confirm requests follow the configured endpoint

### Implementation for User Story 3

- [ ] T021 [US3] Ensure all chat requests use `getBackendBaseUrl()` from `frontend/src/config.ts` inside `frontend/src/api/chatClient.ts` (no hardcoded unreachable host)
- [ ] T022 [US3] Document `VITE_BACKEND_URL` setup and restart behavior in `frontend/README.md` and align with `specs/001-agent-chat-app/quickstart.md`

**Checkpoint**: Env-configured backend URL verified end-to-end

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T023 [P] Confirm UTF-8 Traditional Chinese round-trip in UI + stream path (no mojibake) via manual check noted in `frontend/README.md`
- [ ] T024 Run end-to-end validation steps from `specs/001-agent-chat-app/quickstart.md` and fix any gaps in run docs under `backend/README.md` / `frontend/README.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup — BLOCKS all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational — MVP path
- **User Story 2 (Phase 4)**: Depends on Foundational — can proceed in parallel with US1 after T007 exists
- **User Story 3 (Phase 5)**: Depends on US1 chat client existing (T015/T018) for meaningful verification
- **Polish (Phase 6)**: After desired stories complete

### User Story Dependencies

- **US1 (P1)**: After Foundational — no dependency on US2/US3
- **US2 (P2)**: After Foundational — independent of chat UI
- **US3 (P2)**: After US1 client wiring — independently testable by changing env and restarting

### Parallel Opportunities

- T003–T006 in Setup can run in parallel
- T008–T009–T011 in Foundational can run in parallel after T007 starts
- T012 and T015–T016 can proceed in parallel once Foundational completes
- T019 can run in parallel with US1 implementation

---

## Parallel Example: User Story 1

```bash
# After Foundational:
Task: "Implement stub streaming agent in backend/src/agent/stream.py"
Task: "Implement SSE chat client in frontend/src/api/chatClient.ts"
Task: "Build message list UI in frontend/src/components/ChatThread.tsx"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE** streamed chat independently
5. Add US2 health + US3 env config, then polish

### Incremental Delivery

1. Setup + Foundational → foundation ready
2. US1 streamed chat → demo MVP
3. US2 health endpoint → operability check
4. US3 env-configured frontend URL → AC complete
5. Polish / quickstart validation

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] labels map to spec user stories US1–US3
- No automated test tasks (not requested in spec)
- Commit after each task or logical group
- Stop at checkpoints to validate independently
