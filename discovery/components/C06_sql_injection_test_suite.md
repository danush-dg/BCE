## C06 — SQL injection test suite
ID: M-011
Layer: infra
Source file: api/test_injection.py

---

**Module** — SQL injection test suite
**ID** — M-011
**Layer** — infra
**Primary Responsibility** — Automated verification that five canonical SQL injection payloads are neutralised by M-005's parameterised query (INV-04). Confirms that payloads produce 404 (not 500 or 200) and that the database row count is unchanged after all requests.

**Inputs**
- `BASE_URL` environment variable — defaults to `http://localhost:8000`
- `API_KEY` environment variable — defaults to `""` (empty string)
- `POSTGRES_USER` environment variable — defaults to `postgres` (used in `docker compose exec psql`)
- `POSTGRES_DB` environment variable — defaults to `customer-risk-api`
- Requires the running stack AND docker CLI available in PATH (for `docker compose exec`)

**Outputs**
6 unittest test cases:

| Test | What it asserts |
|---|---|
| `test_payload_1_drop_table` | `'; DROP TABLE customer_risk_profiles; --` → status 404 (not 500, not 200) |
| `test_payload_2_or_true` | `' OR '1'='1` → status 404 |
| `test_payload_3_union_select` | `' UNION SELECT null, null, null --` → status 404 |
| `test_payload_4_stacked_query` | `1; SELECT * FROM customer_risk_profiles` → status 404 |
| `test_payload_5_long_keyword_string` | 500-char repeated SQL keyword string → status 404 |
| `test_row_count_unchanged` | `SELECT COUNT(*) FROM customer_risk_profiles` via psql = pre-test count |

`setUpClass` records pre-test row count via `docker compose exec db psql`. `test_row_count_unchanged` compares against it after all payload tests run.

**`_assert_404` helper** — for each payload asserts:
1. `status_code != 500` (would indicate unhandled exception or injection causing a DB error)
2. `status_code != 200` (would indicate injection succeeded in matching or returning a row)
3. `status_code == 404`

**Public Interface**
```
python -m pytest api/test_injection.py
# or
python api/test_injection.py
```
Requires `BASE_URL`, `API_KEY`, `POSTGRES_USER`, `POSTGRES_DB` in environment.

**Error Behaviour**
- `_get_row_count()` calls `docker compose exec db psql` via subprocess. If docker CLI is unavailable or the db container is not running, `int(result.stdout.strip())` raises `ValueError` (empty stdout) — `setUpClass` fails and all 6 tests error.
- Payloads are URL-path components (not query params). FastAPI's path parameter parsing and URL encoding affect what the API receives. `encodeURIComponent` is applied in the browser JS (M-004), but not in this test suite — the payload strings are passed to `requests.get` directly and `requests` handles URL encoding in its path.

**Known Fragility**
- `subprocess.run(["docker", "compose", "exec", ...], cwd=os.path.dirname(os.path.dirname(__file__)))` — assumes docker CLI and `docker compose` plugin are available in PATH. Fails if run in a Docker-in-Docker environment without host socket access.
- `cwd` is set to `os.path.dirname(os.path.dirname(__file__))` — the project root containing docker-compose.yml. If the test file is moved, this path breaks.
- `POSTGRES_USER` and `POSTGRES_DB` defaults (`postgres`, `customer-risk-api`) may not match the actual `.env` values — if `.env` uses different values, row count queries fail.
- Test suite does not verify the content of any 404 response body — only the status code. A 404 with unexpected body content (e.g., framework error detail) would pass these tests.

**Change Impact**
- If M-005's parameterised query is changed to use string formatting, payloads may produce 500 (DB error) or 200 (injection match), failing these tests.
- If the `customer_risk_profiles` table is renamed or seed data is modified, `_get_row_count()` still works (it queries by table name) but COUNT changes must be accounted for in `test_row_count_unchanged`.

**Invariants verified**
IC-4 (parameterised queries — SQL injection prevention), IC-3 (no write operations — row count check)

**Callers** — engineer or CI runner
**Calls** — None (HTTP requests to M-005, subprocess calls to docker compose exec — neither is a module call)
**Integration Points Used** — None
