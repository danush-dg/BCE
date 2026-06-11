STAGE-2-COMPLETE: SESSION E — 2026-06-11 — BCE Stage 2 Session E (CD)

# INTEGRATION_CONTRACTS.md — Customer Risk API
Produced by: BCE Stage 2 Session E (CD) — 2026-06-11
Sources: discovery/TOPOLOGY.md (A03), discovery/MODULE_CONTRACTS.md,
discovery/INVARIANT_CATALOGUE.md, discovery/INTAKE_SUMMARY.md,
discovery/RISK_REGISTER.md, api/main.py

---

## IP-001 — Postgres 15

Called by: M-002 (get_db_connection — direct), M-005 (get_customer — via M-002)

---

**What the application promises to send:**

Single parameterised SELECT query, issued via psycopg2 cursor on every authenticated
request to GET /customers/{customer_id}:

```sql
SELECT customer_id, risk_tier, risk_factors
FROM customer_risk_profiles
WHERE customer_id = %s
```

- Query is a static string literal — no f-string, no concatenation, no formatting.
- `customer_id` path parameter passed as positional tuple `(customer_id,)` — psycopg2
  handles parameterisation; SQL injection is structurally prevented (IC-4).
- No INSERT, UPDATE, DELETE, or DDL is issued through any application code path (IC-3).
- Connection opened per request via `psycopg2.connect()` with five env-var parameters:
  POSTGRES_HOST, POSTGRES_PORT, POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD.
  No connection pool. No persistent connection state between requests (IC-1, IC-14).
- Connection closed unconditionally in `finally` block at api/main.py:104 (IC-14).

---

**What the application assumes it will receive:**

On row found (customer_id match):
- One row: `(VARCHAR customer_id, VARCHAR risk_tier, TEXT[] risk_factors)`
- psycopg2 maps TEXT[] to a Python list automatically (default psycopg2 behaviour —
  no explicit type adapter registered; confirmed by IC-5 passing in M-012).
- risk_tier value is one of `LOW`, `MEDIUM`, `HIGH` — enforced by CHECK constraint in
  db/init.sql; the application does not re-validate the value after retrieval (IC-6
  depends on DB-layer enforcement at M-007).
- Row content returned verbatim as `{"customer_id": row[0], "risk_tier": row[1],
  "risk_factors": row[2]}` — no transformation, no default substitution.

On no row found (customer_id not in table):
- `cur.fetchone()` returns `None`.
- Application raises HTTPException(404, "Customer not found") — IC-2.

On exception (connection failure, env var missing, query error):
- Any exception from psycopg2.connect() or cursor operations is caught by
  `except Exception:` in M-002 (api/main.py:76).
- HTTPException(500, "Internal server error") raised — static literal, no detail
  propagated (IC-9).
- Note: exceptions raised inside the `try` block in M-005 (api/main.py:95–103) after
  `get_db_connection()` succeeds are not caught by M-002 — they propagate to FastAPI's
  default 500 handler. The only exception raised inside this try block is HTTPException(404),
  which FastAPI handles correctly. No bare exceptions exist inside this try block.

---

**Auth mechanism:**

Postgres username/password supplied via environment variables (POSTGRES_USER,
POSTGRES_PASSWORD) injected from `.env` via docker-compose.yml `env_file: .env`.
Credentials are never hardcoded in application source (confirmed: no literal strings
matching credential patterns in api/main.py or docker-compose.yml).
Postgres connection is on Docker Compose internal network — not a public endpoint from
the application's perspective.

Note: `ports: 5432:5432` in docker-compose.yml exposes Postgres on the host network at
port 5432. Any process on the host can connect directly using valid Postgres credentials,
bypassing the application entirely. See R-006.

---

**Error handling assumptions:**

The application assumes all errors from IP-001 are transient or fatal-at-startup and
does not attempt retry, circuit-breaking, or graceful degradation:

- Connection failure at request time → M-002 catches exception → HTTP 500 with static
  literal. No retry. Caller receives 500 and must retry at the HTTP layer if desired.
- Postgres unavailable at startup → Docker Compose `service_healthy` gate prevents API
  container from starting until db container is healthy (5 retries × 5s = 25s max).
  If all retries fail, API never starts (STARTUP-FATAL — M-008).
- Missing env vars → `os.environ["KEY"]` raises KeyError → caught by `except Exception:`
  in M-002 → HTTP 500. Application cannot start in a degraded state.
- Connection limit exhaustion → psycopg2.OperationalError → caught by `except Exception:`
  in M-002 → HTTP 500. No application-level detection of this condition (R-004).

---

**Known divergences:**

None. The A03 contract fields are consistent with MODULE_CONTRACTS.md M-002/M-005 and
with the invariant enforcement points in INVARIANT_CATALOGUE.md. No divergence between
documented design intent and observed code behaviour for this integration point.

One structural note (not a divergence — a gap in A03's original framing): A03 described
"all psycopg2 exceptions caught." Code uses `except Exception:` which is broader. This
was flagged as STAGE-2-UPDATE in TOPOLOGY.md and confirmed as intentional in M-002
(narrowing to `except psycopg2.Error` would leave KeyError unhandled). The broader
handler is the correct implementation given the design constraints.

---

**Gaps:**

1. **Host port exposure (Postgres bypass):** `ports: 5432:5432` in M-008 exposes Postgres
   on the host at port 5432. A caller with valid Postgres credentials can read or write
   `customer_risk_profiles` directly, bypassing IC-3 (no writes through API), IC-4
   (parameterised queries), IC-7 (auth before DB), IC-8 (key not in response), IC-9
   (error isolation), and all application-layer invariants. Not relevant on a developer
   workstation; P2 for any shared-host or production deployment. → R-006.

2. **No automated regression for IC-14 (connection closure):** The `finally: conn.close()`
   clause is not covered by any test in M-010, M-011, or M-012. A refactor removing or
   misplacing the `finally` block would not be caught by automated tests. Under concurrent
   load, unreleased connections exhaust Postgres `max_connections` and produce 500s. → R-007.

3. **psycopg2 TEXT[] mapping assumption unverified explicitly:** The application assumes
   psycopg2 maps TEXT[] to a Python list without registering an explicit type adapter.
   This is correct for standard psycopg2 behaviour, but no test explicitly asserts on
   the Python type of `risk_factors` before serialisation. IC-5 passing (response shape)
   provides implicit coverage. NOT DETERMINABLE FROM SOURCE whether this is considered a
   gap by the team — annotation candidate (P3).