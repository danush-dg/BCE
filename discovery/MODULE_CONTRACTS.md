STAGE-1-DRAFT: DOCS-DERIVED — 2026-06-11 — Produced by BCE Adapter Pipeline Stage 1
STAGE-2-COMPLETE: SESSIONS B AND C — 2026-06-11 — BCE Stage 2 Sessions B and C (CC)

# MODULE_CONTRACTS.md — Customer Risk API
Stage 1 produced by: BCE Adapter Pipeline Stage 1 (CC)
Stage 2 completed by: BCE Stage 2 Sessions B and C (CC) — 2026-06-11
Sources (Stage 2): api/main.py, db/init.sql, docker-compose.yml, api/Dockerfile,
api/test_errors.py, api/test_injection.py, api/test_invariants.py

All S1-NN skeleton IDs replaced with permanent M-NNN IDs from discovery/ID_REGISTRY.md.
Component files: discovery/components/B01–B05 (serving), discovery/components/C01–C07 (infra).

---

## M-001 — verify_api_key

**Module** — verify_api_key
**ID** — M-001
**Layer** — serving
**Source file** — api/main.py:63–66
**Primary Responsibility** — Validate the `X-API-Key` request header against the `API_KEY` environment variable. Raise HTTPException(401, "Unauthorized") for any invalid, missing, or empty key before any route body executes.

**Inputs**
- `api_key: str` — value from `X-API-Key` header via `Header(None, alias="X-API-Key")`. `None` if header absent; empty string if header present but blank.
- `API_KEY` environment variable — read via `os.environ.get("API_KEY")`. Returns `None` if unset.

**Outputs**
- Returns `None` on success — route body executes.
- Raises `HTTPException(status_code=401, detail="Unauthorized")` on any failure. Static string literal — not an f-string, not derived from the submitted key.

**Public Interface**
`async def verify_api_key(api_key: str = Header(None, alias="X-API-Key")) -> None`
Declared as `Depends(verify_api_key)` on M-005 only. Not on M-003 or M-004.

**Error Behaviour**
All failure paths (missing, empty, wrong key) reach a single `raise` statement — INV-12 enforced structurally. `os.environ.get("API_KEY")` (not `os.environ["API_KEY"]`) ensures a missing env var produces 401s, not 500s (fail-safe).

**Known Fragility**
- Single raise enforces INV-12 structurally; refactoring into separate conditionals would require explicit INV-12 re-verification.
- Per-route injection: new routes added without `Depends(verify_api_key)` are unauthenticated (R-005). CLAUDE.md prohibits new routes within PBVI scope.

**Change Impact**
Changing `detail` violates execution_plan.md permitted literals and INV-12. Adding logging of `api_key` or `valid_key` violates INV-08. Splitting condition into separate raises violates INV-12.

**Callers** — M-005 (via `Depends(verify_api_key)`)
**Calls** — None
**Integration Points Used** — None

---

## M-002 — get_db_connection

**Module** — get_db_connection
**ID** — M-002
**Layer** — serving
**Source file** — api/main.py:69–79
**Primary Responsibility** — Open a psycopg2 connection to Postgres using environment variable credentials. Return the connection on success. Catch all exceptions and raise HTTPException(500, "Internal server error") — never propagating exception detail.

**Inputs**
Five env vars via `os.environ["KEY"]` (KeyError if missing — caught by `except Exception`):
`POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`

**Outputs**
- `psycopg2.connection` on success — caller (M-005) is responsible for closing in a `finally` block.
- `HTTPException(status_code=500, detail="Internal server error")` on any exception. Static literal — no `str(e)`, no traceback fragment.

**Public Interface**
`def get_db_connection() -> psycopg2.connection`
Called directly at api/main.py:94 by M-005.

**Error Behaviour**
`except Exception:` (bare — no binding) covers psycopg2.OperationalError, psycopg2.ProgrammingError, KeyError from missing env vars, and all other exceptions. Intentionally broad — narrowing to `except psycopg2.Error` would expose KeyError as an unhandled exception, breaking INV-09. No `finally` in this function — `psycopg2.connect()` is atomic.

**Known Fragility**
- `except Exception:` bare — exception object explicitly discarded. No path to accidentally propagate detail.
- No connection pooling (R-004). CLAUDE.md prohibits adding a pool.
- `port=os.environ["POSTGRES_PORT"]` passes port as string — psycopg2 accepts str or int.

**Change Impact**
Narrowing `except Exception` to a specific type risks INV-09 violation (highest-severity regression in this system). Adding f-string or `str(e)` to detail violates INV-09. Any detail string change violates execution_plan.md permitted literals.

**Callers** — M-005 (direct call at api/main.py:94)
**Calls** — IP-001 (Postgres 15 via psycopg2.connect())
**Integration Points Used** — IP-001

---

## M-003 — health route handler

**Module** — health route handler
**ID** — M-003
**Layer** — serving
**Source file** — api/main.py:82–84
**Primary Responsibility** — Return `{"status": "ok"}` with no database interaction, no authentication, and no external dependencies.

**Inputs**
None.

**Outputs**
`{"status": "ok"}` — static Python dict literal, serialised to JSON. HTTP 200. Exactly one key (INV-13).

**Public Interface**
`GET /health` — registered on M-006, no auth dependency.

**Error Behaviour**
No error paths. Static dict literal with no IO.

**Known Fragility**
Adding any field to the return dict violates INV-13. Adding a DB call violates INV-13 and potentially INV-09.

**Change Impact**
Adding fields → INV-13. Adding DB call → INV-09 + INV-13. Adding auth dependency → violates CLAUDE.md scope boundary.

**Callers** — M-006 (framework dispatch), HTTP client
**Calls** — None
**Integration Points Used** — None

---

## M-004 — root (UI) route handler

**Module** — root (UI) route handler
**ID** — M-004
**Layer** — serving
**Source file** — api/main.py:87–89 (handler); api/main.py:7–58 (_HTML constant)
**Primary Responsibility** — Return an HTMLResponse containing the complete browser UI as the `_HTML` module-level string constant. Unauthenticated.

**Inputs**
None. `_HTML` is a module-level constant — no runtime input affects the response.

**Outputs**
`HTMLResponse(content=_HTML)` — HTTP 200, Content-Type: text/html. Static string defined at lines 7–58.

**Public Interface**
`GET /` — registered on M-006 with `response_class=HTMLResponse`, no auth dependency.

**INV-11 Enforcement — CONFIRMED FROM SOURCE**
JavaScript `lookup()` function renders `data.customer_id`, `data.risk_tier`, `data.risk_factors` directly. No switch statements, no label maps, no conditional display logic on tier or factor values. `data.risk_factors.map(f => '  - ' + f)` applies uniform bullet formatting — factor strings themselves rendered verbatim. Fetch target is `/customers/` + `encodeURIComponent(id)` — same-origin, no external call (INV-10 safe from JS).

**Error Behaviour**
Server-side: none. Client-side (JS): network failure → `'Request failed.'`; API error → `data.detail`; empty ID → `'Enter a customer ID.'`

**Known Fragility**
- `data.risk_factors.map(...)` throws TypeError if `risk_factors` is non-array — silent failure via catch block.
- `_HTML` is a Python string — JS syntax errors are silent at build time.
- `if (data.detail)` check couples to API response shape — any future 200 response with a `detail` field would render the error path.

**Change Impact**
Adding JS transformation of tier values or factor filtering → INV-11. Adding CDN script tag → INV-10 + CLAUDE.md fixed stack. Adding auth → violates CLAUDE.md scope boundary.

**Callers** — M-006 (framework dispatch), HTTP client (browser)
**Calls** — None
**Integration Points Used** — None

---

## M-005 — get_customer route handler

**Module** — get_customer route handler
**ID** — M-005
**Layer** — serving
**Source file** — api/main.py:92–106
**Primary Responsibility** — Authenticate the request, open a psycopg2 connection, execute a parameterised SELECT against `customer_risk_profiles`, and return the customer risk tier record as JSON. Return 404 for unknown IDs. Close the connection unconditionally.

**Inputs**
- `customer_id: str` — URL path parameter. No format validation — arbitrary strings safe via parameterised query.
- `api_key: str = Depends(verify_api_key)` — M-001 resolved before route body executes.

**Outputs**
- **200**: `{"customer_id": row[0], "risk_tier": row[1], "risk_factors": row[2]}` — exactly 3 keys; `risk_factors` is a Python list (psycopg2 maps TEXT[] automatically).
- **404**: `{"detail": "Customer not found"}`
- **401**: `{"detail": "Unauthorized"}` — raised by M-001 before body executes.
- **500**: `{"detail": "Internal server error"}` — raised by M-002 before `try` block.

**Public Interface**
`GET /customers/{customer_id}` with `Depends(verify_api_key)` — registered on M-006.

**SQL Query**
`SELECT customer_id, risk_tier, risk_factors FROM customer_risk_profiles WHERE customer_id = %s` — static literal, tuple parameter `(customer_id,)`. INV-04 enforced structurally.

**Error Behaviour**
- `conn = get_db_connection()` outside `try` block — if M-002 raises, `finally: conn.close()` is never entered (no connection to close, no NameError).
- `finally: conn.close()` runs on HTTPException(404), normal return, and any exception inside the `try` block — no connection leak path.
- Exception from `cur.execute()`/`cur.fetchone()` after successful connect propagates to FastAPI default handler — 500 with framework-generated body (not the controlled static literal).

**Known Fragility**
- **Positional column access** `row[0]`, `row[1]`, `row[2]` — bound to SELECT column order. Column reordering silently transposes values in the 200 response (INV-05/INV-06 violation with no exception).
- No query timeout — slow DB query holds uvicorn worker thread indefinitely.
- `api_key: str = Depends(verify_api_key)` parameter is unused in the body — it is the dependency injection hook. Removing it removes the auth gate.

**Change Impact**
Highest blast-radius module. Touches INV-01, INV-02, INV-04, INV-05, INV-06, INV-07, INV-09. Column reordering in SELECT silently breaks INV-05/INV-06.

**Callers** — M-006 (framework dispatch), HTTP client
**Calls** — M-001 (via Depends), M-002 (direct call)
**Integration Points Used** — IP-001 (indirectly via M-002)

---

## M-006 — FastAPI application bootstrap

**Module** — FastAPI application bootstrap
**ID** — M-006
**Layer** — infra
**Source file** — api/main.py:60
**Primary Responsibility** — Instantiate the FastAPI ASGI application object (`app = FastAPI()`). All route registrations, dependency injection wiring, and framework dispatch operate through this object.

**Inputs**
`FastAPI()` with no arguments — default settings. No middleware, no custom exception handlers, no CORS.

**Outputs**
`app` — FastAPI ASGI application instance; module-level global. Referenced by M-009 as `main:app`.

**Error Behaviour**
STARTUP-FATAL if FastAPI is not installed. No custom exception handlers — unhandled exceptions in route bodies produce FastAPI default 500 (not the system's controlled static literals).

**Known Fragility**
No global `Depends(verify_api_key)` — auth is per-route. A future route added without auth is unauthenticated (R-005). This is the place to add a global auth default with `/health` and `/` excluded.

**Change Impact**
Adding middleware/exception handlers affects all routes globally. Renaming `app` breaks M-009's uvicorn CMD.

**Callers** — M-009 (uvicorn `main:app`)
**Calls** — M-003, M-004, M-005 (framework dispatch)
**Integration Points Used** — None

---

## M-007 — database initialisation

**Module** — database initialisation
**ID** — M-007
**Layer** — infra
**Source file** — db/init.sql
**Primary Responsibility** — Define the `customer_risk_profiles` table schema and insert 9 seed rows (3 per tier). Executed once by Postgres at container initialisation. Not a runtime module.

**Inputs**
Executed by Postgres 15 entrypoint on first start (empty data volume). No application input.

**Outputs**
`customer_risk_profiles` table: `customer_id VARCHAR PK`, `risk_tier VARCHAR NOT NULL CHECK('LOW','MEDIUM','HIGH')`, `risk_factors TEXT[] NOT NULL`. 9 seed rows: CUST-001 through CUST-009.

**Error Behaviour**
`CREATE TABLE IF NOT EXISTS` — idempotent. INSERTs are not idempotent (duplicate PK error if re-run against existing table, but this cannot occur normally).

**Known Fragility**
- `docker compose down` without `-v` leaves volume — init.sql does not re-run on next up. Stale seed data is silent (R-003).
- `CHECK` constraint enforces INV-06 at DB layer — weakening it removes the data-layer guard.
- No migration system — schema changes require manual ALTER TABLE or volume clear.

**Change Impact**
Column changes require matching updates in M-005 SELECT query (INV-04), response dict (INV-05), and M-010/M-012 test assertions. Removing CHECK constraint weakens INV-06 to application-level only.

**Callers** — Postgres Docker image entrypoint (not the application)
**Calls** — None
**Integration Points Used** — None

---

## M-008 — container orchestration

**Module** — container orchestration
**ID** — M-008
**Layer** — infra
**Source file** — docker-compose.yml
**Primary Responsibility** — Declare the two-container stack, health-based startup sequencing, port mappings, and environment injection. Single source of truth for how the system is assembled.

**Inputs**
`.env` file injected via `env_file: .env` on both services. `db/init.sql` mounted as volume. `./api/` directory built as api image.

**Outputs**
`db` service: postgres:15, ports 5432:5432, pg_isready healthcheck (interval: 5s, retries: 5).
`api` service: built from ./api, ports 8000:8000, `depends_on: db: condition: service_healthy`.

**Error Behaviour**
db healthcheck fails all 5 retries → api service never starts (STARTUP-FATAL). No `start_period` — retries begin immediately.

**Known Fragility**
- [STAGE-2-DIVERGENCE — 2026-06-11]: Stage 1 noted "no external networks" as isolation evidence. Code also shows `ports: 5432:5432` on db service — Postgres accessible on host port 5432 directly, bypassing API auth and all application invariants. Development-scope risk; P2 for any shared-host or production deployment. [RESOLVED — 2026-06-11]: CONFIRMED. Stage 1 was correct but incomplete. Finding accepted; captured in RISK_REGISTER.md as R-006 (P2).
[ENGINEER NOTE — 2026-06-11]: `ports: 5432:5432` on the db service is accepted for development scope. This system is a single-developer internal ops tool (per INTAKE_SUMMARY.md). Direct Postgres access on host port 5432 is acceptable in this deployment context. Before any shared-host or production deployment, `ports: 5432:5432` must be removed from docker-compose.yml — R-006 tracks this as a P2 action item. No code change made at this time. Resolves P1-S3-002.
- M-012 TestInv09 calls `docker compose stop db` / `docker compose start db` — if tearDownClass fails, db container remains stopped until manual recovery.
- No external network definitions — INV-10 confirmed at Compose layer.

**Change Impact**
Removing `condition: service_healthy` risks cold-start connection errors (INV-02 violation window). Adding `external: true` network violates INV-10. Adding a third service (e.g., Redis) requires new depends_on and would violate INV-01 if used for response caching.

**Callers** — Docker Compose CLI (engineer, CI)
**Calls** — None
**Integration Points Used** — None

---

## M-009 — container image definition

**Module** — container image definition
**ID** — M-009
**Layer** — infra
**Source file** — api/Dockerfile
**Primary Responsibility** — Define the reproducible build for the api container. Establishes Python 3.11 runtime, installs dependencies, copies source, declares the uvicorn startup command.

**Inputs**
`api/requirements.txt` (copied first for layer caching); `api/` directory contents (copied after pip install).

**Outputs**
Container image: `python:3.11-slim` base, `/app` workdir, packages from requirements.txt installed, source copied, `EXPOSE 8000`, `CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]` (exec form).

**Error Behaviour**
`RUN pip install` — STARTUP-FATAL if any package fails to install. No version pins — package resolution depends on PyPI state at build time.

**Known Fragility**
- **No version pins in requirements.txt** — highest-impact silent fragility in the infra layer. A breaking upstream release produces a broken image without any local change.
- Container runs as root — no `USER` instruction.
- `COPY . .` copies test files into the production image — dead weight, not harmful.
- Single uvicorn worker — consistent with per-request psycopg2 connection design (R-004).

**Change Impact**
Changing `main:app` in CMD requires matching rename in main.py. Adding `--workers` changes concurrency model and interacts with per-request DB connections. Adding version pins makes builds reproducible (strongly recommended).

**Callers** — M-008 (`build: ./api`)
**Calls** — M-006 (uvicorn `main:app`)
**Integration Points Used** — None

---

## M-010 — error state test suite

**Module** — error state test suite
**ID** — M-010
**Layer** — infra
**Source file** — api/test_errors.py
**Primary Responsibility** — Automated verification of auth failure states (401) and not-found state (404). Confirms INV-08 (partial) and INV-12 against the live API.

**Inputs**
`BASE_URL` (default: http://localhost:8000), `API_KEY` (default: ""). Requires running stack. No direct DB access.

**Outputs**
5 test cases: no key → 401, wrong key → 401, empty key → 401, CUST-999 → 404, identical 401 bodies.

**Known Fragility**
`API_KEY` default is `""` — `test_nonexistent_customer_returns_404` and `test_401_bodies_are_identical` fail if `API_KEY` env var is not set to the correct value.

**Invariants verified** — IC-7, IC-8 (partial), IC-9 (partial), IC-12

**Callers** — engineer or CI
**Calls** — None
**Integration Points Used** — None

---

## M-011 — SQL injection test suite

**Module** — SQL injection test suite
**ID** — M-011
**Layer** — infra
**Source file** — api/test_injection.py
**Primary Responsibility** — Automated verification that 5 SQL injection payloads produce 404 (not 500 or 200) and that row count is unchanged after all requests. Verifies INV-04 against the live stack.

**Inputs**
`BASE_URL`, `API_KEY`, `POSTGRES_USER`, `POSTGRES_DB`. Requires running stack and docker CLI in PATH.

**Outputs**
6 test cases: 5 payload → 404 assertions, 1 row count unchanged assertion via `docker compose exec db psql`.

**Known Fragility**
`subprocess` + docker CLI dependency. `cwd` path derived from `__file__` — breaks if test file is moved. `POSTGRES_DB` default may not match `.env`.

**Invariants verified** — IC-3 (partial), IC-4

**Callers** — engineer or CI
**Calls** — None
**Integration Points Used** — None

---

## M-012 — full invariant verification suite

**Module** — full invariant verification suite
**ID** — M-012
**Layer** — infra
**Source file** — api/test_invariants.py
**Primary Responsibility** — Automated regression verification for 11/13 declared invariants. Requires live stack and docker CLI. Includes TestInv09 which stops and restarts the `db` container.

**Inputs**
`BASE_URL`, `API_KEY`, `POSTGRES_USER`, `POSTGRES_DB`. Requires running stack and docker CLI in PATH.

**Outputs**
13 test classes covering INV-01–INV-09, INV-12, INV-13. INV-10 and INV-11 are manual checks only — no automated coverage.

**Known Fragility**
- TestInv09 destructive lifecycle: `docker compose stop db` → tests → `docker compose start db`. If tearDownClass fails, db remains stopped until manual recovery.
- `_get_db_row()` uses f-string SQL interpolation — safe for trusted test literals, not for untrusted input.
- `time.sleep(2)` / `time.sleep(10)` in TestInv09 — fixed delays that may be insufficient on slow hosts.
- **INV-10 and INV-11 not automated** — changes introducing external calls or JS transformation are not caught by this suite.

**Invariants verified** — IC-1, IC-2, IC-3, IC-4, IC-5, IC-6, IC-7, IC-8, IC-9, IC-12, IC-13 (11/13)
**Invariants NOT verified (manual)** — IC-10 (code review + grep), IC-11 (code review + browser devtools)

**Callers** — engineer or CI (Session 5 final gate: 34/34 tests passing — VERIFICATION_RECORD.md)
**Calls** — None
**Integration Points Used** — None

---

## Stage 2 Divergences

### STAGE-2-DIVERGENCE — M-008 — Port 5432 exposure — RESOLVED 2026-06-11
Stage 1 described container isolation by noting "no external networks defined." Code inspection reveals `ports: 5432:5432` on the `db` service — Postgres is accessible directly on host port 5432, bypassing all application-layer auth and invariants. Not an error in Stage 1 (Stage 1 correctly described no external networks), but a gap in the isolation picture. Flagged as P2 risk for any production or shared-host deployment.

**Resolution:** CONFIRMED — finding stands. Stage 1 was incomplete, not wrong. Both facts are true: no external Docker networks defined, AND host port 5432 is exposed, creating a direct-access side-channel outside the API auth boundary. Accepted for development scope. Captured in RISK_REGISTER.md as R-006 (P2). No changes required to source code at this time — action deferred to a hardening enhancement before any production deployment. Pre-Stage 3 sign-off: complete.

### Stage 2 Confirmations (NOT divergences — design intent matched code exactly)
- M-001: single raise for all 401 paths — INV-12 structural enforcement confirmed
- M-002: `except Exception` broader than psycopg2-specific — intentional fail-safe confirmed
- M-004: JavaScript renders values without transformation — INV-11 confirmed
- M-005: `conn = get_db_connection()` outside try block — no connection leak on M-002 raise confirmed
- M-007: `CREATE TABLE IF NOT EXISTS` idempotency confirmed
- M-008: `condition: service_healthy` startup gate confirmed
- M-012: INV-10/INV-11 manual-only coverage confirmed (noted in docs/execution_plan.md)
