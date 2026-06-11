## A02 — Module Call Map
Produced by: BCE Stage 2 Session A (CC)
Date: 2026-06-11
Source files read: api/main.py, db/init.sql, docker-compose.yml, api/Dockerfile, api/requirements.txt,
api/test_errors.py, api/test_injection.py, api/test_invariants.py

---

## Module Roster — Customer Risk API
Generated: 2026-06-11 by BCE Stage 2 Session A (CC)
Note: these IDs are permanent. Do not reassign at later sessions.

| ID | Module Name | Source File | Layer |
|---|---|---|---|
| M-001 | verify_api_key | api/main.py:63–66 | serving |
| M-002 | get_db_connection | api/main.py:69–79 | serving |
| M-003 | health route handler | api/main.py:82–84 | serving |
| M-004 | root (UI) route handler | api/main.py:87–89 | serving |
| M-005 | get_customer route handler | api/main.py:92–106 | serving |
| M-006 | FastAPI application bootstrap | api/main.py:60 | infra |
| M-007 | database initialisation | db/init.sql | infra |
| M-008 | container orchestration | docker-compose.yml | infra |
| M-009 | container image definition | api/Dockerfile | infra |
| M-010 | error state test suite | api/test_errors.py | infra |
| M-011 | SQL injection test suite | api/test_injection.py | infra |
| M-012 | full invariant verification suite | api/test_invariants.py | infra |

This roster is the canonical ID-to-module mapping for all subsequent sessions.
Sessions B, C, D, and E must reference modules by M-NNN in all relationship fields — never by prose name.

UI_SURFACE.md cross-reference: docs/UI_SURFACE.md is not present for this project. The inline HTML UI
(the _HTML constant at api/main.py:7–58) is served by M-004 (root route handler) as a serving-layer
module — not classified as a separate page/layout/component module. No UI_SURFACE.md reconciliation step applies.

---

## Section 1 — Internal Call Table

| Edge | Call Site (file:line) | Sync/Async |
|---|---|---|
| M-006 --[CALLS]--> M-003 | api/main.py:82 (framework dispatch via @app.get("/health") decorator) | S |
| M-006 --[CALLS]--> M-004 | api/main.py:87 (framework dispatch via @app.get("/") decorator) | S |
| M-006 --[CALLS]--> M-005 | api/main.py:92 (framework dispatch via @app.get("/customers/{customer_id}") decorator) | S |
| M-005 --[CALLS]--> M-001 | api/main.py:93 (`api_key: str = Depends(verify_api_key)` — FastAPI dependency injection, resolved before route body executes) | A |
| M-005 --[CALLS]--> M-002 | api/main.py:94 (`conn = get_db_connection()` — direct synchronous call, only reached after M-001 resolves without raising) | S |

Notes:
- M-006 CALLS M-003/M-004/M-005 via framework dispatch, not direct Python function calls. Route handlers are registered at module load time (decorator execution). FastAPI resolves registered routes at runtime per request.
- M-005 → M-001 is async: verify_api_key is `async def` (line 63). FastAPI's dependency injection awaits it before executing the route body.
- M-005 → M-002 is sync: get_db_connection is `def` (line 69), called directly. FastAPI runs synchronous handlers in a thread pool.
- M-003 (health) and M-004 (root) make no calls to other modules.
- Test modules (M-010, M-011, M-012) interact with the system via HTTP requests to the running API — not direct Python calls. They are not part of the runtime call graph.

---

## Section 2 — Startup Sequence

| Step | Module (M-NNN) | Action | Failure Mode |
|---|---|---|---|
| 1 | M-008 | Docker Compose starts `db` service (postgres:15) | STARTUP-FATAL — stack cannot continue if Postgres image fails to start |
| 2 | M-007 | Postgres executes db/init.sql via `/docker-entrypoint-initdb.d/` on first start (volume empty) | STARTUP-FATAL for schema creation failure; silent skip if volume exists (data persists from prior run) |
| 3 | M-008 | Docker Compose runs `pg_isready` healthcheck on `db` service every 5s, up to 5 retries | NON-FATAL per retry; STARTUP-FATAL if all 5 retries fail — `api` service never starts |
| 4 | M-008 | Docker Compose starts `api` service once `db` condition: service_healthy passes | STARTUP-FATAL — api blocked until db healthy |
| 5 | M-009 | Container image starts; uvicorn launches; Python imports api/main.py | STARTUP-FATAL — import error if fastapi, psycopg2-binary, or uvicorn not installed |
| 6 | M-006 | `app = FastAPI()` instantiates the application object (line 60) | STARTUP-FATAL — all route registration depends on `app` |
| 7 | M-001 | `verify_api_key` function defined (line 63) | NON-FATAL as definition; will fail at request time if API_KEY env var unset (returns None, causing all keys to fail) |
| 8 | M-002 | `get_db_connection` function defined (line 69) | NON-FATAL as definition; fails at request time if POSTGRES_* env vars missing (KeyError caught by except Exception → 500) |
| 9 | M-006 | `@app.get("/health")` registers M-003 (line 82) | STARTUP-FATAL if route registration fails |
| 10 | M-006 | `@app.get("/")` registers M-004 (line 87) | STARTUP-FATAL if route registration fails |
| 11 | M-006 | `@app.get("/customers/{customer_id}")` registers M-005 with Depends(verify_api_key) (line 92) | STARTUP-FATAL if route or dependency registration fails |
| 12 | M-006 | Uvicorn starts listening on 0.0.0.0:8000 | STARTUP-FATAL — no requests served until this completes |

Key observation: No database connection is opened at startup. The first psycopg2 connection is opened on the first authenticated request to GET /customers/{customer_id}. A missing or wrong POSTGRES_* env var produces a 500 on first request, not a startup failure.

---

## Section 3 — Async Boundaries

| Producer (M-NNN) | Consumer (M-NNN) | Mechanism | Failure behaviour |
|---|---|---|---|
| M-005 (get_customer) | M-001 (verify_api_key) | FastAPI async dependency injection — `Depends(verify_api_key)` awaited before route body | M-001 raises HTTPException(401, "Unauthorized"); M-005 body never executes; 401 returned to client |

Notes:
- All route handlers (M-003, M-004, M-005) are synchronous (`def`, not `async def`). FastAPI runs them in a thread pool via `asyncio.to_thread` semantics. This is not an async boundary in the producer/consumer sense — it is a framework concurrency mechanism.
- M-002 (get_db_connection) and its psycopg2 calls are entirely synchronous (blocking). No async/await in the DB access path.
- No message queues, background tasks, or WebSocket connections in this system.
- The only genuine async boundary is the dependency injection mechanism (M-005 → M-001).
