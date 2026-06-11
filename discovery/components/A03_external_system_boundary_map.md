## A03 — External System Boundary Map
Produced by: BCE Stage 2 Session A (CC)
Date: 2026-06-11
Source files read: api/main.py, docker-compose.yml, db/init.sql

Stage 1 A03 sourced from docs/architecture.md fixed stack and integration descriptions.
Stage 2 confirms all error handling, auth, and request/response fields from source code.
IP-NNN IDs assigned in this session and registered in discovery/ID_REGISTRY.md.

---

## IP-001 — Postgres 15 (db container)

| Field | Value |
|---|---|
| **IP-NNN** | IP-001 |
| **System name** | Postgres 15 |
| **System type** | Relational database |
| **Role** | Authoritative data store — pre-assessed customer risk tier records |
| **Location** | Internal to Docker Compose stack — `db` service on Compose default bridge network. Not an external network call (INV-10 confirmed). |
| **Connection mechanism** | `psycopg2.connect()` called in M-002 (get_db_connection) at api/main.py:71–77. Five keyword arguments: `host=os.environ["POSTGRES_HOST"]`, `port=os.environ["POSTGRES_PORT"]`, `dbname=os.environ["POSTGRES_DB"]`, `user=os.environ["POSTGRES_USER"]`, `password=os.environ["POSTGRES_PASSWORD"]`. All five are required — missing any raises `KeyError`, caught by `except Exception` → HTTPException(500). |
| **Auth mechanism** | Postgres username/password credentials injected via environment variables at runtime. Credentials are never hardcoded in source. Auth is handled by psycopg2 handshake — not a separate auth step in application code. |
| **Operations performed** | Single parameterised `SELECT` statement (read-only). Executed via `cur.execute()` at api/main.py:97–100. No INSERT, UPDATE, DELETE, or DDL is reachable through any API path. |
| **Request schema** | `SELECT customer_id, risk_tier, risk_factors FROM customer_risk_profiles WHERE customer_id = %s` — static string literal. `customer_id` value passed as positional tuple `(customer_id,)` — no string formatting or concatenation at any point. |
| **Response schema** | **Found:** `(customer_id VARCHAR, risk_tier VARCHAR, risk_factors TEXT[])` — one-tuple row accessed as `row[0]`, `row[1]`, `row[2]`. **Not found:** empty cursor, `row = None`. psycopg2 maps `TEXT[]` to a Python list automatically. |
| **Error handling** | `except Exception` at api/main.py:78 — broad catch, covers psycopg2.OperationalError, psycopg2.ProgrammingError, KeyError (missing env vars), and any other exception raised during connect. Raises `HTTPException(status_code=500, detail="Internal server error")`. Static literal confirmed — not an f-string, no `str(e)`, no exception detail propagated. Connection closed in `finally` block at api/main.py:105 regardless of outcome. |
| **Retry behaviour** | None — no retry logic in application code. A failed connection raises 500 immediately. |
| **Credentials in response** | Never — INV-08 and INV-09 prohibit any internal detail (including connection strings, hostnames, or credentials) in any HTTP response. |
| **Schema initialisation** | `db/init.sql` executed once by Postgres Docker image on first container start via `/docker-entrypoint-initdb.d/`. Creates `customer_risk_profiles` table and inserts 9 seed rows. `CREATE TABLE IF NOT EXISTS` prevents error on re-run when volume is not cleared. |
| **Called by** | M-002 (get_db_connection), M-005 (get_customer route handler) — M-005 calls M-002 which opens the connection. Direct call chain: M-005 → M-002 → IP-001. |
| **Startup dependency** | M-008 (docker-compose.yml) enforces `condition: service_healthy` via `pg_isready` healthcheck (5s interval, 5 retries) before M-009/M-006 start. No application-level connection pool or connection check at startup — first psycopg2 connection is on first authenticated request. |

**No other external systems exist in this project.**

Confirmed from api/main.py import list: `os`, `psycopg2`, `fastapi` only — no `requests`, `httpx`, `urllib.request`, `boto3`, or any external HTTP client. INV-10 (no network calls outside Compose stack) is structurally confirmed by source code inspection.

docker-compose.yml defines no `external: true` network blocks — both services run on the Compose default bridge network.

---

## External System Summary

| IP-NNN | System Name | Type | Location | Called By | Operations |
|---|---|---|---|---|---|
| IP-001 | Postgres 15 | Relational database | db container (Compose internal) | M-002, M-005 (via M-002) | SELECT only |

Total external systems: 1
Total IP-NNN assigned: IP-001
Next IP-NNN to assign: IP-002
