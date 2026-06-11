## C05 — error state test suite
ID: M-010
Layer: infra
Source file: api/test_errors.py

---

**Module** — error state test suite
**ID** — M-010
**Layer** — infra
**Primary Responsibility** — Automated verification of authentication failure states (401) and customer-not-found state (404). Confirms that INV-08 (key never echoed) and INV-12 (identical 401 bodies) are met by the running API.

**Inputs**
- `BASE_URL` environment variable — defaults to `http://localhost:8000`
- `API_KEY` environment variable — defaults to `""` (empty string)
- Both read via `os.environ.get()` at module load time — defaults apply if not set

Requires a running API stack at `BASE_URL`. Does not interact with Postgres directly.

**Outputs**
5 unittest test cases — all must pass for the suite to be green:

| Test | What it asserts |
|---|---|
| `test_no_api_key_returns_401` | GET /customers/CUST-001 with no X-API-Key header → status 401, detail "Unauthorized" |
| `test_wrong_api_key_returns_401` | GET /customers/CUST-001 with X-API-Key: wrong-key → status 401, detail "Unauthorized" |
| `test_empty_api_key_returns_401` | GET /customers/CUST-001 with X-API-Key: "" → status 401, detail "Unauthorized" |
| `test_nonexistent_customer_returns_404` | GET /customers/CUST-999 with valid key → status 404, detail "Customer not found" |
| `test_401_bodies_are_identical` | No key vs wrong key → `r_no_key.json() == r_wrong_key.json()` (INV-12) |

**Public Interface**
```
python -m pytest api/test_errors.py
# or
python api/test_errors.py
```
Requires `BASE_URL` and `API_KEY` set in environment (or defaults).

**Error Behaviour**
- If the API stack is not running: all tests fail with `requests.exceptions.ConnectionError` — not informative assertion failures. Engineers must start the stack before running tests.
- `API_KEY` default is `""` (empty string). An empty API_KEY causes `test_nonexistent_customer_returns_404` and `test_401_bodies_are_identical` to fail because M-001 rejects the empty key with 401 instead of passing auth. The env var must be set to the correct key for these tests to exercise the authenticated path.

**Known Fragility**
- `API_KEY = os.environ.get("API_KEY", "")` — default is empty string. If `API_KEY` env var is not set, `test_nonexistent_customer_returns_404` will fail (empty key → 401, not the expected 404).
- Tests make real HTTP requests to a live stack — they are integration tests, not unit tests. Flakiness is possible if the stack is under load or if port 8000 is occupied.
- No teardown — tests do not reset any server state. Stateless by nature (read-only API).

**Change Impact**
- If M-001's 401 detail string changes, `test_no_api_key_returns_401`, `test_wrong_api_key_returns_401`, and `test_empty_api_key_returns_401` fail.
- If M-005's 404 detail string changes, `test_nonexistent_customer_returns_404` fails.
- If INV-12 is violated (M-001 emits different bodies for missing vs wrong key), `test_401_bodies_are_identical` fails.
- INV-10 (no external calls) and INV-11 (UI fidelity) are not covered by this suite — manual checks only.

**Invariants verified**
IC-7 (auth gate), IC-8 (key not echoed — partial, via body check), IC-9 (static error literals — partial, for 401/404 paths), IC-12 (identical 401 bodies)

**Callers** — engineer or CI runner
**Calls** — None (HTTP requests to M-005 via requests library — external HTTP, not module calls)
**Integration Points Used** — None
