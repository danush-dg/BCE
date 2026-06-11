STAGE-1-DRAFT: DOCS-DERIVED — 2026-06-11 — Produced by BCE Adapter Pipeline Stage 1
STAGE-2-UPDATE: A01/A02/A03 — 2026-06-11 — BCE Stage 2 Session A (CC)

# TOPOLOGY.md — Customer Risk API
Produced by: BCE Adapter Pipeline Stage 1 (CC) | Updated by: BCE Stage 2 Session A (CC)
Sources (Stage 1): docs/architecture.md, docs/CLAUDE.md, docs/execution_plan.md
Sources (Stage 2 Session A): api/main.py, db/init.sql, docker-compose.yml

---

## A01 — Layer Boundary Map

### Boundary 1 — HTTP Client → FastAPI application (api container, port 8000)

| Field | Value |
|---|---|
| Produced by | HTTP client (browser, curl, or downstream tool integration) |
| Artifact | HTTP GET request |
| Consumed by | FastAPI route dispatcher — routes to health handler, UI handler, or customer lookup handler based on path |
| Handoff mechanism | HTTP over Docker Compose port mapping (host port 8000 → container port 8000) |
| Contract | Three routes only: `GET /health` (no auth), `GET /` (no auth), `GET /customers/{customer_id}` (X-API-Key required). No POST, PUT, PATCH, or DELETE routes exist or may be added within PBVI scope. |
| Failure mode | [STAGE-2-CONFIRMED 2026-06-11] Unhandled exceptions produce FastAPI default 500. All observed error paths are handled: 401 via Depends() guard, 404 via explicit check, 500 via except Exception in get_db_connection. No unhandled code path detected. |

### Boundary 2 — FastAPI auth dependency → customer lookup route handler (internal)

| Field | Value |
|---|---|
| Produced by | FastAPI dependency injection system invoking `verify_api_key` |
| Artifact | Resolved dependency result (no exception = valid key; HTTPException 401 = invalid or missing key) |
| Consumed by | Customer lookup route handler body — only executes if dependency does not raise |
| Handoff mechanism | FastAPI `Depends()` — synchronous gate evaluated before route body |
| Contract | Auth is checked before any database interaction. On failure, 401 is returned with static literal `"Unauthorized"`. No key echo; no partial-match feedback. |
| Failure mode | [STAGE-2-CONFIRMED 2026-06-11] Confirmed from main.py:63–66. Condition is `if not api_key or api_key != valid_key`. `not api_key` catches None (missing) and empty string; `!= valid_key` is exact match. Both cases raise identical HTTPException(401, "Unauthorized"). Static literal confirmed — not an f-string. `os.environ.get("API_KEY")` used, not `os.environ["API_KEY"]` — unset env var returns None, causing all keys to fail auth (not an exception). |

### Boundary 3 — FastAPI customer lookup handler → Postgres (db container)

| Field | Value |
|---|---|
| Produced by | Customer lookup route handler via `get_db_connection()` |
| Artifact | psycopg2 connection + parameterised SELECT cursor result |
| Consumed by | Postgres 15 (db container) executing query against `customer_risk_profiles` |
| Handoff mechanism | psycopg2 TCP connection on internal Docker Compose network (`POSTGRES_HOST=db`, `POSTGRES_PORT=5432`) |
| Contract | Static SELECT query with single `%s` parameter. Read-only. One row (found) → 200 response. No row → 404. Exception → 500 with static literal. |
| Failure mode | [STAGE-2-UPDATE 2026-06-11] Stage 1 described "all psycopg2 exceptions caught". Code uses `except Exception` (main.py:76) — broader than psycopg2.Error alone. This covers KeyError from missing POSTGRES_* env vars too. Confirmed: static literal "Internal server error" in HTTPException detail (not an f-string, no str(e)). `finally: conn.close()` confirmed at main.py:104 — no connection leak path. |

### Boundary 4 — Docker Compose startup dependency (db → api)

| Field | Value |
|---|---|
| Produced by | Docker Compose service health machinery |
| Artifact | `service_healthy` condition on `db` service |
| Consumed by | `api` service startup — blocked until `db` reports healthy |
| Handoff mechanism | `pg_isready` healthcheck (interval 5s, retries 5) on db container |
| Contract | API container does not start until Postgres is accepting connections. Eliminates connection errors at cold start. |
| Failure mode | [STAGE-2-CONFIRMED 2026-06-11] Confirmed from docker-compose.yml. `depends_on: db: condition: service_healthy`. If all 5 retries fail (25s), api service never starts. No fallback — STARTUP-FATAL. No database connection is opened at api startup; first connection is on first authenticated request to GET /customers/{customer_id}. |

---

## A02 — Module Call Map

Source file: discovery/components/A02_module_call_map.md (produced 2026-06-11 by BCE Stage 2 Session A)

### Module Roster

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

IDs are permanent. Do not reassign. Sessions B, C, D, E must use M-NNN identifiers.

### Internal Call Table (summary)

| Edge | Call Mechanism | Sync/Async |
|---|---|---|
| M-006 --[CALLS]--> M-003 | @app.get("/health") framework dispatch | S |
| M-006 --[CALLS]--> M-004 | @app.get("/") framework dispatch | S |
| M-006 --[CALLS]--> M-005 | @app.get("/customers/{customer_id}") framework dispatch | S |
| M-005 --[CALLS]--> M-001 | FastAPI Depends() — resolved before route body (main.py:93) | A |
| M-005 --[CALLS]--> M-002 | Direct function call (main.py:94) — only after M-001 resolves | S |

---

## Stage 2 Completeness Summary — 2026-06-11
Produced by: BCE Stage 2 Sessions A0–E (CC/CD)

| Artifact | Status | NOT DETERMINABLE count | Divergences from Stage 1 | Notes |
|---|---|---|---|---|
| TOPOLOGY.md | COMPLETE | 0 | 0 | A01/A02/A03 all filled; Stage 2 update applied |
| MODULE_CONTRACTS.md | COMPLETE | 0 | 1 — RESOLVED | M-001–M-012 full contracts; 1 divergence (M-008 port exposure) confirmed and signed off |
| INTEGRATION_CONTRACTS.md | COMPLETE | 1 | N/A — Stage 2 only | 1 NOT DETERMINABLE entry (annotation candidate) |
| INVARIANT_CATALOGUE.md | COMPLETE | 1 | 0 | IC-1–IC-14; 1 NOT DETERMINABLE entry (annotation candidate) |
| RISK_REGISTER.md | COMPLETE | 2 | 0 | R-001–R-008; 2 NOT DETERMINABLE entries (annotation candidates) |

Stage-2-Divergence items requiring engineer resolution before Stage 3:
- **M-008 — Port 5432 exposure** (MODULE_CONTRACTS.md Known Fragility + Stage 2 Divergences section)
  Status: RESOLVED 2026-06-11 — CONFIRMED. Stage 1 was correct but incomplete. Finding accepted; captured as R-006 (P2) in RISK_REGISTER.md. No source code change required at this time.

NOT DETERMINABLE entries remaining (annotation candidates for Stage 3 / engineer annotation pass):
- INTEGRATION_CONTRACTS.md — 1 entry
- INVARIANT_CATALOGUE.md — 1 entry
- RISK_REGISTER.md — 2 entries
Total: 4 entries. All are annotation candidates requiring human operational knowledge — not CC extraction gaps.

Sessions completed: A0 (CC), A (CC), F01 (CC), F02 (CC), F03 (CD), B+C (CC), D (CC), E (CD)
Component files: A00, A02, A03, B01–B05, C01–C07 (15 files in discovery/components/)
Domain model: DOMAIN_MODEL.json produced (non-sparse); ID_REGISTRY.md updated with E-NNN, A-NNN, SV-NNN, SVV-NNN IDs
A01 component file gap: A01_layer_boundary_map.md not produced as a standalone file — A01 content lives in TOPOLOGY.md only. Convention gap, not content gap. May surface as P3 annotation item in Stage 3.

Human gate required before Stage 3 begins.
Engineer sign-off: _____________________________ Date: _______________

Full call table, startup sequence, and async boundary detail: discovery/components/A02_module_call_map.md

---

## A03 — External System Boundary Map

### External System 1 — Postgres 15 (db container)

**Integration Point ID: IP-001** [ASSIGNED 2026-06-11 — BCE Stage 2 Session A]
Called by: M-002 (get_db_connection) and M-005 (get_customer) indirectly via M-002

| Field | Value |
|---|---|
| System name | Postgres 15 |
| Role | Authoritative data store — pre-assessed customer risk tier data |
| Location | Internal to Docker Compose stack — `db` service on Compose internal network. Not an external network call (INV-10). |
| Connection mechanism | `psycopg2.connect()` with five environment variable parameters: POSTGRES_HOST, POSTGRES_PORT, POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD |
| Auth mechanism | Postgres username/password from environment variables. Credentials never hardcoded in application source. |
| Operations performed | Single parameterised SELECT: `SELECT customer_id, risk_tier, risk_factors FROM customer_risk_profiles WHERE customer_id = %s`. No INSERT, UPDATE, DELETE, or DDL reachable through any API path. |
| Request schema | Static SQL string with single `%s` placeholder. `customer_id` value passed as positional parameter tuple — no string formatting or concatenation. Confirmed in main.py:97–100. |
| Response schema | One row (found): `(customer_id VARCHAR, risk_tier VARCHAR, risk_factors TEXT[])`. No row (not found): empty cursor. Confirmed in main.py:101–106. |
| Error handling | [STAGE-2-CONFIRMED 2026-06-11] `except Exception` in main.py:76 — broader than psycopg2.Error, catches all exceptions including env var KeyError. HTTPException(500, "Internal server error") raised. No exception detail propagated. Connection closed in `finally` block (main.py:104). |
| Credentials in response | Never — INV-08 and INV-09 prohibit any internal detail in responses |
| Called by | M-002 (get_db_connection), M-005 (get_customer) via M-002 |
| IP-NNN | IP-001 |

**Note:** No other external systems exist. INV-10 (Operational Isolation) mandates that the system makes no network calls outside the Docker Compose stack. Confirmed from main.py: no `import requests`, `import httpx`, or `import urllib` statements. No external network definitions in docker-compose.yml.
