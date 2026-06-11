STAGE-1-DRAFT: DOCS-DERIVED — 2026-06-11 — Produced by BCE Adapter Pipeline Stage 1
STAGE-2-COMPLETE: SESSION D — 2026-06-11 — BCE Stage 2 Session D (CC)

# INVARIANT_CATALOGUE.md — Customer Risk API
Stage 1 produced by: BCE Adapter Pipeline Stage 1 (CC)
Stage 2 completed by: BCE Stage 2 Session D (CC) — 2026-06-11
Sources (Stage 2): api/main.py, db/init.sql, docker-compose.yml, discovery/MODULE_CONTRACTS.md,
discovery/components/A02_module_call_map.md, docs/invariant.md

Stage 2 changes:
- All 13 "Currently enforced: NOT DETERMINABLE FROM SOURCE" fields resolved from source code
- Enforcement point, Owning module, Enforcing modules fields added to all IC records
- IC-14 added (code-observed — not in docs/invariant.md)
- No Stage 1 IC records removed or contradicted

---

## IC-1 — Data Correctness (No Caching)

**ID:** IC-1
**Statement:** The API reads from the database on every request. Values returned must match the database row exactly. No caching layer is permitted between the API and the database.
**Category:** Data Correctness
**Scope:** GLOBAL
**Why it matters:** An operations staff member makes decisions based on the risk tier returned. A cached or transformed value produces decisions on false data.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer — `conn = get_db_connection()` called on every request with no memoisation, no response cache, no `@functools.lru_cache`; `FastAPI()` constructed with no arguments — no cache middleware registered (api/main.py:60)
**Owning module:** M-005
**Enforcing modules:** M-005, M-006
**Source:** docs/invariant.md INV-01; docs/CLAUDE.md INV-01
**Invariant touch map:** S1.T2, S3.T2, S5.T2 (TestInv01DataCorrectness in M-012)

---

## IC-2 — Existence Mapping (200 or 404 Only)

**ID:** IC-2
**Statement:** For any value of `customer_id`, the only valid outcomes are 200 (found) or 404 (not found). No other status code is valid for a request that reaches lookup logic with a valid API key. A missing customer must never return 200 with empty data, null fields, or a default tier.
**Category:** Operational
**Scope:** GLOBAL
**Why it matters:** A non-existent customer ID that returns 200 can be confused with a real customer. A downstream tool treating an empty or default response as authoritative may classify a missing customer as low-risk.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer:101–103 — `row = cur.fetchone(); if row is None: raise HTTPException(status_code=404, detail="Customer not found")`. No other return path in the route body. 401 and 500 are pre-body failures (M-001 and M-002 respectively), not reachable through the lookup logic with a valid key.
**Owning module:** M-005
**Enforcing modules:** M-005
**Source:** docs/invariant.md INV-02; docs/CLAUDE.md INV-02
**Invariant touch map:** S3.T1, S4.T2, S5.T2 (TestInv02StatusCodes in M-012)

---

## IC-3 — No Write Operations

**ID:** IC-3
**Statement:** The API executes no write operations against the database. No INSERT, UPDATE, DELETE, or DDL statement is reachable through any API endpoint under any input. The database state after any sequence of API calls must be identical to its state before.
**Category:** Security
**Scope:** GLOBAL
**Why it matters:** A write vector through the API would allow corruption or destruction of seed data, removing the reliability of all subsequent API responses.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer:97–100 — the sole `cur.execute()` call issues `SELECT customer_id, risk_tier, risk_factors FROM customer_risk_profiles WHERE customer_id = %s` — read-only; api/main.py:app — M-006 registers only three `@app.get()` routes (GET only). No POST, PUT, PATCH, or DELETE route decorators exist anywhere in main.py.
**Owning module:** M-005
**Enforcing modules:** M-005, M-006
**Source:** docs/invariant.md INV-03; docs/CLAUDE.md INV-03
**Invariant touch map:** S5.T1 (M-011 row count), S5.T2 (TestInv03NoWrites in M-012)

---

## IC-4 — Parameterised Queries (SQL Injection Prevention)

**ID:** IC-4
**Statement:** The entire query is a static string with a single `%s` placeholder. The `customer_id` path parameter is passed to psycopg2 as a query parameter tuple. No part of the query is constructed from user input via string formatting or concatenation.
**Category:** Security
**Scope:** GLOBAL
**Why it matters:** A crafted `customer_id` value can execute arbitrary SQL if the query is constructed by string formatting.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer:97–100 — `cur.execute("SELECT customer_id, risk_tier, risk_factors FROM customer_risk_profiles WHERE customer_id = %s", (customer_id,))`. The first argument is a string literal (no f-string, no `%` or `.format()` call, no concatenation). `customer_id` is passed as the second argument — a positional tuple `(customer_id,)`.
**Owning module:** M-005
**Enforcing modules:** M-005
**Source:** docs/invariant.md INV-04; docs/CLAUDE.md INV-04
**Invariant touch map:** S3.T1, S5.T1 (M-011), S5.T2 (TestInv04InjectionBlocked in M-012)

---

## IC-5 — Response Shape (Exactly Three Fields)

**ID:** IC-5
**Statement:** Every 200 response contains exactly three fields: `customer_id` (string), `risk_tier` (string), and `risk_factors` (array). The shape is consistent regardless of tier value or factor list length. No additional fields. No missing fields. No variation in key names or value types.
**Category:** Data Correctness
**Scope:** GLOBAL
**Why it matters:** A downstream tool built against this contract breaks on an unexpected shape.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer:104 — `return {"customer_id": row[0], "risk_tier": row[1], "risk_factors": row[2]}`. Static dict literal with exactly three keys. `risk_factors` (row[2]) is a Python list — psycopg2 automatically maps `TEXT[]` to list at api/main.py:96 (`cur = conn.cursor()`). FastAPI serialises the Python list to a JSON array.
**Owning module:** M-005
**Enforcing modules:** M-005
**Source:** docs/invariant.md INV-05; docs/CLAUDE.md INV-05
**Invariant touch map:** S1.T2, S3.T1, S5.T2 (TestInv05ResponseShape in M-012)

---

## IC-6 — Risk Tier Values (Controlled Vocabulary)

**ID:** IC-6
**Statement:** `risk_tier` is always one of three uppercase literal values: `LOW`, `MEDIUM`, or `HIGH`. The value is passed through from the database without transformation. No case variation is permitted.
**Category:** Data Correctness
**Scope:** GLOBAL
**Why it matters:** A downstream tool with a three-value enum breaks on an unrecognised tier.
**Currently enforced:** YES — dual-layer enforcement
**Enforcement points:**
- db/init.sql:4 — `CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH'))` — DB-layer constraint prevents any row with an out-of-vocabulary tier from existing
- api/main.py:get_customer:104 — `row[1]` passed through directly; no conditional transformation, no `.lower()`, no label map
**Owning module:** M-007 (authoritative enforcement — CHECK constraint at data layer)
**Enforcing modules:** M-007, M-005
**Source:** docs/invariant.md INV-06; docs/CLAUDE.md INV-06
**Invariant touch map:** S1.T2, S3.T1, S5.T2 (TestInv06RiskTierValues in M-012)

---

## IC-7 — Authentication Gate (Auth Before DB)

**ID:** IC-7
**Statement:** `GET /customers/{customer_id}` requires a valid API key in the `X-API-Key` header. Requests without a valid key return 401 before any database interaction occurs. `GET /health` and `GET /` are intentionally unauthenticated. No other routes exist.
**Category:** Security
**Scope:** TASK-SCOPED — applies to GET /customers/{customer_id}
**Why it matters:** Any path that executes DB operations before auth checking exposes customer data to unauthenticated callers.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer:93 — `api_key: str = Depends(verify_api_key)` is declared as a function parameter. FastAPI's dependency injection system resolves all `Depends()` before executing the route body. M-001 (verify_api_key) raises HTTPException(401) before the route body line `conn = get_db_connection()` (api/main.py:94) is ever reached. The auth-before-DB ordering is enforced by FastAPI's framework semantics, not by application code ordering alone.
**Owning module:** M-001
**Enforcing modules:** M-001, M-005, M-006
**Source:** docs/invariant.md INV-07; docs/CLAUDE.md INV-07
**Verification note:** The "before DB interaction" clause is confirmed by: (1) code review of parameter declaration ordering, (2) FastAPI framework guarantee that Depends() is resolved pre-body, (3) TestInv07AuthBeforeDB in M-012 (proxy check — unauthenticated request returns 401 and row count is unchanged). Code review is the definitive verification.
**Invariant touch map:** S2.T4, S3.T1, S4.T2, S5.T2 (TestInv07AuthBeforeDB in M-012)

---

## IC-8 — API Key Never Exposed

**ID:** IC-8
**Statement:** The API key never appears in any HTTP response body, header, or log output. The 401 detail string is a static literal. It does not echo the submitted key, does not confirm what a valid key looks like, and does not vary based on the submitted value.
**Category:** Security
**Scope:** GLOBAL
**Why it matters:** A caller who submits a wrong key must receive no information about the correct key. Echoing the submitted value confirms receipt and narrows the attack surface.
**Currently enforced:** YES
**Enforcement point:** api/main.py:verify_api_key:66 — `raise HTTPException(status_code=401, detail="Unauthorized")` — static string literal, not an f-string, does not reference `api_key` or `valid_key`. No `print()`, `logging.`, or any logging call anywhere in main.py (confirmed by full source read — zero logging statements).
**Owning module:** M-001
**Enforcing modules:** M-001
**Source:** docs/invariant.md INV-08; docs/CLAUDE.md INV-08
**Verification note:** HTTP response body confirmed by M-010 TestInv08 and M-012 TestInv08KeyNotInResponse. Log output can only be confirmed by runtime log inspection — cannot be proven by static code analysis alone (no logging framework configured, but uvicorn access logs are outside application code scope).
**Invariant touch map:** S2.T4, S4.T2, S5.T2 (TestInv08KeyNotInResponse in M-012)

---

## IC-9 — No Internal Detail in Responses

**ID:** IC-9
**Statement:** No internal implementation detail appears in any HTTP response. Only permitted error response bodies are pre-defined static strings: `"Unauthorized"`, `"Customer not found"`, `"Internal server error"`.
**Category:** Security
**Scope:** GLOBAL
**Why it matters:** A caller who triggers a database error and receives a traceback gains the schema, connection parameters, and technology stack.
**Currently enforced:** YES
**Enforcement points:**
- api/main.py:get_db_connection:78-79 — `except Exception: raise HTTPException(status_code=500, detail="Internal server error")`. Bare `except Exception:` (no `as e` binding) — exception object is explicitly discarded. Static literal. Covers all exception types including psycopg2 errors and KeyError from missing env vars.
- api/main.py:verify_api_key:66 — `detail="Unauthorized"` — static literal
- api/main.py:get_customer:103 — `detail="Customer not found"` — static literal
- FastAPI default exception handler: any unhandled exception in route handlers produces FastAPI's own 500 response (not the controlled literal). No unhandled code path currently exists in M-003, M-004, or M-005, but this is a per-route responsibility, not globally enforced.
**Owning module:** M-002 (500 path — primary risk; highest-consequence failure mode)
**Enforcing modules:** M-001, M-002, M-005
**Source:** docs/invariant.md INV-09; docs/CLAUDE.md INV-09; docs/execution_plan.md Permitted Error Literals table
**Invariant touch map:** S2.T3, S2.T4, S3.T3, S5.T2 (TestInv09InternalErrorIsolation in M-012)

---

## IC-10 — Operational Isolation (No External Network Calls)

**ID:** IC-10
**Statement:** The system makes no network calls to any address outside the Docker Compose stack. The API container communicates only with the `db` container on the internal Compose network. No DNS lookups, no HTTP calls, no external logging, no telemetry.
**Category:** Operational
**Scope:** GLOBAL
**Why it matters:** Customer risk data could be transmitted to an external endpoint, breaking the confidentiality guarantee.
**Currently enforced:** YES
**Enforcement points:**
- api/main.py import list — only `os`, `psycopg2`, `fastapi.*` modules imported. No `requests`, `httpx`, `urllib.request`, `aiohttp`, `boto3`, or any outbound HTTP client. Confirmed by full source read.
- docker-compose.yml — no `networks:` block with `external: true`. Both services on Compose default bridge network.
- api/main.py:root — `_HTML` JavaScript uses `fetch('/customers/' + encodeURIComponent(id), ...)` — relative URL (same-origin), not an absolute external URL. No CDN links in `_HTML`.
**Owning module:** M-006 (application-level isolation — no external imports)
**Enforcing modules:** M-006, M-008, M-004
**Source:** docs/invariant.md INV-10; docs/CLAUDE.md INV-10
**Verification note:** INV-10 has no automated test in M-012 — manual check only: grep api/main.py for `requests`, `httpx`, `urllib` (absent) and inspect docker-compose.yml for `external:` (absent). Any future dependency addition to requirements.txt must be reviewed against this invariant.
**Invariant touch map:** S1.T3, S5.T2 (manual — TestInv10 not implemented in M-012)

---

## IC-11 — UI Fidelity (No Transformation)

**ID:** IC-11
**Statement:** The UI renders the API response without transformation. The values displayed for `customer_id`, `risk_tier`, and `risk_factors` are the values returned by the API. The JavaScript does not remap tier strings, does not reorder or filter factors, does not apply display labels that differ from the API values, and does not add or remove content.
**Category:** Data Correctness
**Scope:** TASK-SCOPED — applies to GET / (M-004) and embedded JavaScript
**Why it matters:** An operations staff member sees a tier label that does not match what the API returned, or a factor is hidden. The UI becomes a source of misrepresentation.
**Currently enforced:** YES — confirmed by code review of _HTML constant
**Enforcement point:** api/main.py:root — _HTML constant (lines 7–58); JavaScript `lookup()` function (lines 31–52 of _HTML). Rendering code:
```
out.textContent =
  'customer_id: ' + data.customer_id + '\n' +
  'risk_tier: ' + data.risk_tier + '\n' +
  'risk_factors:\n' + data.risk_factors.map(function(f) { return '  - ' + f; }).join('\n');
```
`data.customer_id` and `data.risk_tier` concatenated directly — no conditional branching on their values. `data.risk_factors.map(f => '  - ' + f)` applies uniform bullet prefix — factor strings themselves rendered verbatim. No switch statements, no `{ LOW: 'Low Risk' }` maps, no `.filter()` calls.
**Owning module:** M-004
**Enforcing modules:** M-004
**Source:** docs/invariant.md INV-11; docs/CLAUDE.md INV-11
**Verification note:** INV-11 has no automated test in M-012 — manual check only: code review of _HTML confirmed above; definitive runtime check requires browser devtools DOM comparison (fetch API response directly, compare field values to rendered DOM). [STAGE-2-CONFIRMED — 2026-06-11]: code review confirms no transformation.
**Invariant touch map:** S4.T1, S5.T2 (manual — TestInv11 not implemented in M-012)

---

## IC-12 — 401 Response Is a Static Literal

**ID:** IC-12
**Statement:** The 401 response body is a single static literal regardless of whether the key was missing, empty, or incorrect. The response does not vary based on the nature of the authentication failure.
**Category:** Security
**Scope:** TASK-SCOPED — applies to verify_api_key (M-001) and all auth failure paths
**Why it matters:** Varying 401 messages reveal information about the authentication mechanism.
**Currently enforced:** YES — enforced structurally by a single raise statement
**Enforcement point:** api/main.py:verify_api_key:65-66 — single condition `if not api_key or api_key != valid_key:` with a single `raise HTTPException(status_code=401, detail="Unauthorized")`. All three failure modes (missing, empty, wrong key) reach the same raise statement — identical response guaranteed by code structure, not by runtime comparison.
**Owning module:** M-001
**Enforcing modules:** M-001
**Source:** docs/invariant.md INV-12; docs/CLAUDE.md INV-12
**Invariant touch map:** S2.T4, S4.T2 (test_401_bodies_are_identical in M-010), S5.T2 (TestInv12IdenticalUnauthorizedBodies in M-012)

---

## IC-13 — Health Endpoint Returns No Data

**ID:** IC-13
**Statement:** The `/health` endpoint returns only `{"status": "ok"}`. No customer data, no database row counts, no schema information, no internal system detail.
**Category:** Security
**Scope:** TASK-SCOPED — applies to GET /health (M-003)
**Why it matters:** The unauthenticated `/health` endpoint would become an information leak if it exposed any database state.
**Currently enforced:** YES — enforced structurally by a static dict literal
**Enforcement point:** api/main.py:health:83-84 — `return {"status": "ok"}`. Single-key static dict literal. No DB calls. No calls to any other module. The function body is two lines: the decorator and the return statement.
**Owning module:** M-003
**Enforcing modules:** M-003
**Source:** docs/invariant.md INV-13; docs/CLAUDE.md INV-13
**Invariant touch map:** S2.T2, S5.T2 (TestInv13HealthShape in M-012)

---

## IC-14 — Connection Closed Per Request (code-observed)

**ID:** IC-14
**Statement:** Every psycopg2 connection opened during a `GET /customers/{customer_id}` request is closed exactly once before the HTTP response is returned, regardless of whether the request succeeded, raised HTTPException(404), or encountered an exception inside the try block.
**Category:** Operational
**Scope:** TASK-SCOPED — applies to M-005 (get_customer) only
**Why it matters:** psycopg2 connections are not pooled. Each opened connection occupies a Postgres connection slot (default limit: 100). If a connection is not closed, it persists until the uvicorn worker thread terminates. Under concurrent load, unreleased connections exhaust `max_connections`, causing all subsequent DB requests to fail with 500s — an outage that is not observable from the application code itself.
**Currently enforced:** YES
**Enforcement point:** api/main.py:get_customer:104-105 — `finally: conn.close()`. The `finally` clause is unconditional — it executes on normal return, on HTTPException(404) raised inside the `try` block, and on any other exception raised inside the `try` block. Note: if M-002 (`get_db_connection()`) raises before the `try` block is entered (api/main.py:94), `conn` is never assigned and the `finally` block is never reached — no connection was opened, so there is nothing to close. This is correct behaviour.
**Owning module:** M-005
**Enforcing modules:** M-005
**Source:** Code-observed — not in docs/invariant.md
**Rationale:** This invariant is implicit in the code and was not documented in the project's invariant specification. It is the operational complement of IC-1 (no caching) — IC-1 ensures no DB state is retained across requests at the data layer; IC-14 ensures no DB connections are retained across requests at the resource layer.

---

## Automated Coverage Summary

| IC | Category | Currently enforced | Automated test | Test location |
|---|---|---|---|---|
| IC-1 | Data Correctness | YES | YES | M-012 TestInv01DataCorrectness |
| IC-2 | Operational | YES | YES | M-012 TestInv02StatusCodes |
| IC-3 | Security | YES | YES | M-011 + M-012 TestInv03NoWrites |
| IC-4 | Security | YES | YES | M-011 + M-012 TestInv04InjectionBlocked |
| IC-5 | Data Correctness | YES | YES | M-012 TestInv05ResponseShape |
| IC-6 | Data Correctness | YES | YES | M-012 TestInv06RiskTierValues |
| IC-7 | Security | YES | YES (proxy) | M-012 TestInv07AuthBeforeDB |
| IC-8 | Security | YES | YES (partial) | M-010 test_401_bodies_are_identical + M-012 TestInv08KeyNotInResponse |
| IC-9 | Security | YES | YES | M-012 TestInv09InternalErrorIsolation |
| IC-10 | Operational | YES | **NO — manual only** | Code review + grep |
| IC-11 | Data Correctness | YES | **NO — manual only** | Code review + browser devtools |
| IC-12 | Security | YES | YES | M-010 test_401_bodies_are_identical + M-012 TestInv12IdenticalUnauthorizedBodies |
| IC-13 | Security | YES | YES | M-012 TestInv13HealthShape |
| IC-14 | Operational | YES | **NO — code-observed only** | Not in test suite |

**Coverage gap:** IC-10, IC-11, and IC-14 have no automated regression test. A code change that introduces an external import (IC-10), adds JS transformation logic (IC-11), or removes the `finally: conn.close()` (IC-14) would not be caught by the test suites in M-010, M-011, or M-012.

---

## Stage 2 Divergences

None. All 13 Stage 1 IC records are confirmed exactly as described by docs/invariant.md. No Stage 1 invariant was incorrectly stated or missing from code enforcement.

One addition: IC-14 (code-observed) — not a divergence from Stage 1, as Stage 1 did not claim this invariant was absent. It is a new finding from code walkthrough.
