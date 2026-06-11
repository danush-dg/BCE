## C07 — full invariant verification suite
ID: M-012
Layer: infra
Source file: api/test_invariants.py

---

**Module** — full invariant verification suite
**ID** — M-012
**Layer** — infra
**Primary Responsibility** — Automated regression verification for 11 of 13 declared invariants (INV-01 through INV-09, INV-12, INV-13). Requires a live stack. Includes a destructive sub-test (TestInv09) that stops and restarts the `db` container.

**Inputs**
- `BASE_URL` environment variable — defaults to `http://localhost:8000`
- `API_KEY` environment variable — defaults to `""` (empty string)
- `POSTGRES_USER` environment variable — defaults to `postgres`
- `POSTGRES_DB` environment variable — defaults to `customer-risk-api`
- Requires: running stack, docker CLI in PATH, docker socket accessible

**Outputs**
13 test classes (11 invariants covered — see coverage gap below):

| Class | Invariant | Key assertions |
|---|---|---|
| `TestInv01DataCorrectness` | INV-01 | API response matches DB row exactly for CUST-001, CUST-004, CUST-007 (all three fields) |
| `TestInv02StatusCodes` | INV-02 | CUST-001 → 200; CUST-999 → 404 with "Customer not found" |
| `TestInv03NoWrites` | INV-03 | Row count unchanged after GET requests to CUST-001/004/007/999 |
| `TestInv04InjectionBlocked` | INV-04 | 5 injection payloads → not 500, not 200 |
| `TestInv05ResponseShape` | INV-05 | 200 response has exactly keys `{customer_id, risk_tier, risk_factors}`, correct types |
| `TestInv06RiskTierValues` | INV-06 | risk_tier in `{"LOW", "MEDIUM", "HIGH"}` for CUST-001/004/007 |
| `TestInv07AuthBeforeDB` | INV-07 | No key → 401; row count unchanged (proxy check — code review is the definitive check) |
| `TestInv08KeyNotInResponse` | INV-08 | Submitted key string `probe-key-that-must-not-appear` absent from 401 response text |
| `TestInv09InternalErrorIsolation` | INV-09 | DB stopped → 500 with `{"detail": "Internal server error"}`; body lacks "psycopg2", "customer_risk_profiles", "Traceback" |
| `TestInv12IdenticalUnauthorizedBodies` | INV-12 | Wrong key vs no key → same JSON response body |
| `TestInv13HealthShape` | INV-13 | GET /health → 200, exactly one key `["status"]`, value `"ok"` |

**Invariant coverage gap:**
- **INV-10** (no external network calls) — no automated test. Verified by code review (grep for `import requests`/`httpx`/`urllib` in main.py — absent) and absence of `external: true` in docker-compose.yml.
- **INV-11** (UI fidelity — no transformation in JS) — no automated test. Verified by code review of `_HTML`'s `lookup()` function (B04 contract confirms direct rendering without switch/map logic) and manual browser devtools comparison.

**TestInv09 destructive lifecycle:**
- `setUpClass`: `docker compose stop db` → `time.sleep(2)` (waits for container to be unreachable)
- Tests run: two assertions against a stopped DB
- `tearDownClass`: `docker compose start db` → `time.sleep(10)` (waits for healthcheck recovery)
- **Risk**: if `tearDownClass` does not run (test runner killed, Python crash), the db container remains stopped. All subsequent test runs in the same session fail with unexpected errors. Engineers must manually run `docker compose start db` to recover.

**Helper functions:**
- `_psql(query)` — runs `docker compose exec db psql` with `-t -A -c query`, returns stdout stripped. Used by TestInv01 (_get_db_row), TestInv03/TestInv07 (_get_row_count).
- `_get_db_row(customer_id)` — executes `SELECT ... WHERE customer_id = '{customer_id}'` via f-string in `_psql`. Customer IDs passed are trusted test literals (CUST-001 etc.) — not untrusted input. This is test-only code; the fragility does not exist in production M-005.
- `_get_row_count()` — wraps `_psql("SELECT COUNT(*) FROM customer_risk_profiles")`.

**Public Interface**
```
python -m pytest api/test_invariants.py
# or
python api/test_invariants.py
```

**Known Fragility**
- `_get_db_row` uses an f-string to construct SQL (`WHERE customer_id = '{customer_id}'`). This is intentional for test tooling (only called with trusted literal IDs) but would be a vulnerability if reused with untrusted input.
- `time.sleep(2)` and `time.sleep(10)` in TestInv09 are fixed delays. On a slow host, the db container may not be fully down after 2s or fully up after 10s, causing TestInv09 to flake.
- `POSTGRES_DB` default `customer-risk-api` — if `.env` specifies a different DB name, `_psql` queries fail with "database not found".
- `API_KEY = os.environ.get("API_KEY", "")` default empty — tests that require a valid API key (TestInv01, TestInv02 existing customer, TestInv03, TestInv05, TestInv06, TestInv08 proxy) will fail with 401 if `API_KEY` is not set to the correct value.
- INV-07's TestInv07AuthBeforeDB uses row count as a proxy for "no DB interaction before auth". This cannot definitively prove auth happens before DB interaction — it only shows no write occurred. The definitive verification is code review confirming `Depends(verify_api_key)` is declared before any DB call (confirmed in B05 contract).

**Change Impact**
- Changes to any of M-001 through M-005's error messages break corresponding test assertions.
- Changes to seed data (M-007) break TestInv01 (DB vs API comparison), TestInv02 (specific customer IDs), TestInv05, TestInv06 (specific IDs used).
- INV-10 and INV-11 remain unautomated — any change to main.py that introduces an external call or JS transformation would not be caught by this suite.

**Invariants verified**
IC-1, IC-2, IC-3, IC-4, IC-5, IC-6, IC-7, IC-8, IC-9, IC-12, IC-13 (11/13 — INV-10 and INV-11 are manual)

**Callers** — engineer or CI runner (Session 5 final gate: all 13 classes passing from cold start — verified in docs/VERIFICATION_RECORD.md)
**Calls** — None
**Integration Points Used** — None
