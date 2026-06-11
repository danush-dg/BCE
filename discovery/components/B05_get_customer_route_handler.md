## B05 — get_customer route handler
ID: M-005
Layer: serving
Source file: api/main.py:92–106

---

**Module** — get_customer route handler
**ID** — M-005
**Layer** — serving
**Primary Responsibility** — Authenticate the request, open a psycopg2 connection, execute a parameterised SELECT against `customer_risk_profiles`, and return the customer's risk tier record as JSON. Return 404 for unknown IDs. Close the connection unconditionally.

**Inputs**
- `customer_id: str` — path parameter extracted from URL (`/customers/{customer_id}`). Any string is accepted; FastAPI performs no format validation. Parameterised query (INV-04) ensures arbitrary strings are safe to pass to Postgres.
- `api_key: str = Depends(verify_api_key)` — M-001 resolved by FastAPI before route body executes. If M-001 raises, the route body is never entered.

**Outputs**
- **200 JSON** (found): `{"customer_id": row[0], "risk_tier": row[1], "risk_factors": row[2]}` — exactly three keys, values read directly from psycopg2 cursor tuple. `risk_factors` is a Python list (psycopg2 maps `TEXT[]` automatically).
- **404 JSON** (not found): `{"detail": "Customer not found"}` — raised as `HTTPException(404, "Customer not found")`.
- **401 JSON** (auth failure): `{"detail": "Unauthorized"}` — raised by M-001 before body executes.
- **500 JSON** (DB failure): `{"detail": "Internal server error"}` — raised by M-002 before `try` block.

**Public Interface**
```
GET /customers/{customer_id}
```
Registered on M-006 via `@app.get("/customers/{customer_id}")` at api/main.py:92. Declares `Depends(verify_api_key)` as a route parameter.

**SQL Query** (api/main.py:97–100)
```sql
SELECT customer_id, risk_tier, risk_factors
FROM customer_risk_profiles
WHERE customer_id = %s
```
Static string literal. `customer_id` passed as positional tuple `(customer_id,)`. No string formatting or concatenation at any point. INV-04 enforced structurally — the query string is a literal, not assembled.

**Error Behaviour**

| Failure condition | When it occurs | HTTP response |
|---|---|---|
| Invalid/missing `X-API-Key` | Before route body — M-001 raises | 401 `{"detail": "Unauthorized"}` |
| DB unreachable or env var missing | `conn = get_db_connection()` raises via M-002 — before `try` | 500 `{"detail": "Internal server error"}` |
| Customer ID not in database | `row is None` after `cur.fetchone()` | 404 `{"detail": "Customer not found"}` |
| DB error after connection opens | Exception inside `try` block — not explicitly caught here; propagates to FastAPI | FastAPI default 500 |

**Connection lifecycle:**
- `conn = get_db_connection()` is outside the `try` block (api/main.py:94). If M-002 raises, the `try...finally` is never entered — no connection to close, no NameError.
- `finally: conn.close()` (api/main.py:104–105) runs regardless of whether the `try` body raises `HTTPException(404)` or returns normally. Connection is always closed.

**Known Fragility**
- **Positional column access**: `row[0]`, `row[1]`, `row[2]` — bound to SELECT column order, not column names. If the SELECT list were reordered (e.g., `SELECT risk_tier, customer_id, risk_factors`), values would be silently transposed in the 200 response. INV-05 and INV-06 would be violated without any exception or test failure at the DB layer.
- **No query timeout**: a locked or slow Postgres query holds the uvicorn worker thread indefinitely. Under concurrent load this can exhaust the thread pool. CLAUDE.md fixed stack prohibits async DB drivers that would mitigate this.
- **DB exception inside `try` not explicitly caught**: an exception from `cur.execute()` or `cur.fetchone()` after a successful connect propagates to FastAPI's default exception handler. FastAPI returns 500 but the detail content would be FastAPI's generic message, not the controlled static literal. This path is theoretically reachable if the connection degrades after `get_db_connection()` returns successfully but before the cursor executes.
- **`api_key: str = Depends(verify_api_key)` parameter name**: the route function receives the `api_key` value but does not use it (auth check is inside M-001). This unused parameter is intentional — it is the dependency injection hook. Renaming or removing it would remove the auth gate.

**Change Impact**
Highest blast-radius module in the system. Changes touch:
- INV-01 (data correctness — any caching or transformation introduced here)
- INV-02 (status codes — must remain 200 or 404 for valid-key requests)
- INV-04 (parameterised query — any string formatting in the SQL)
- INV-05 (response shape — exactly three keys)
- INV-06 (risk_tier pass-through — any transformation of row[1])
- INV-07 (auth gate — removing Depends or adding a DB call before it)
- INV-09 (exception isolation — any unhandled exception path after connect)

**Callers** — M-006 (dispatches `GET /customers/{customer_id}` requests); HTTP client (external)
**Calls** — M-001 (verify_api_key via Depends), M-002 (get_db_connection via direct call)
**Integration Points Used** — IP-001 (Postgres 15, indirectly via M-002)
