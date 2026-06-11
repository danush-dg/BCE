STAGE-1-DRAFT: DOCS-DERIVED — 2026-06-11 — Produced by BCE Adapter Pipeline Stage 1
STAGE-2-COMPLETE: SESSION E — 2026-06-11 — BCE Stage 2 Session E (CD)

# RISK_REGISTER.md — Customer Risk API
Produced by: BCE Adapter Pipeline Stage 1 (CC)
Updated by: BCE Stage 2 Session E (CD) — 2026-06-11
Sources (Stage 1): docs/architecture.md (Sections 3–4), docs/CLAUDE.md, docs/VERIFICATION_RECORD.md
Sources (Stage 2): discovery/TOPOLOGY.md, discovery/MODULE_CONTRACTS.md,
discovery/INTEGRATION_CONTRACTS.md, discovery/INVARIANT_CATALOGUE.md, api/main.py,
docker-compose.yml

Stage 2 changes:
- All five Stage 1 "Recommended action: NOT DETERMINABLE FROM SOURCE" fields resolved
- Module and invariant references updated to M-NNN / IC-N identifiers
- R-006, R-007, R-008 added from code-observed gaps

---

## R-001 — psycopg2 Exception Surfaces in HTTP Response

**Risk ID:** R-001
**Description:** If a database connection fails or query raises and is not caught (or is caught too narrowly), the raw psycopg2 traceback reaches the 500 response body, exposing table names, column names, psycopg2 connection strings, and application file paths.
**Severity:** P2
**Source artifact:** docs/architecture.md R-01
**Invariants at risk:** INV-09 — only pre-defined static strings permitted in error responses
**Status:** Mitigated — verified in VERIFICATION_RECORD.md (T2.3 and T3.3): HTTPException(500, "Internal server error") confirmed; psycopg2 detail confirmed absent from response body. Verification test TestInv09InternalErrorIsolation passes from cold start.
**Mitigation (from docs):** `get_db_connection` must wrap all psycopg2 exceptions and return a sanitised 500 with a static error message. Session 3 (T3.3) verified by stopping the db container and confirming response body.
**Affected module:** M-002
**Invariants at risk:** IC-9
**Recommended action:** [STAGE-2-RESOLVED — 2026-06-11] M-002 uses `except Exception:` (bare, no binding) — broader than psycopg2.Error, intentionally. This covers KeyError from missing POSTGRES_* env vars as well as psycopg2 exceptions. Narrowing to `except psycopg2.Error` would leave KeyError unhandled, breaking IC-9. No code change required. Ongoing: any future edit to M-002 exception handler must preserve the broad catch — narrowing is the highest-severity regression path in this codebase.
**Severity rationale:** Downgraded from P1 to P2 because mitigated and verified. Remains P2 (not P3) because this is the highest-consequence failure mode in the system — any regression here immediately violates INV-09 in a security-impactful way.

---

## R-002 — Inline HTML Grows Beyond Maintainability Threshold

**Risk ID:** R-002
**Description:** Embedding HTML and JavaScript in Python source (main.py) couples UI changes to API container rebuilds. As UI complexity grows, the inline string becomes difficult to read, modify, and test safely. An error in the inline string may produce a silent runtime failure (malformed HTML served) rather than a build-time error.
**Severity:** P3
**Source artifact:** docs/architecture.md R-02; DD-01 Challenge verdict
**Affected module:** M-004
**Invariants at risk:** IC-11
**Status:** Accepted — scope boundary in docs/CLAUDE.md limits UI to one text input and one result display. No additional states or interactivity permitted within PBVI scope.
**Mitigation (from docs):** Scope boundary in CLAUDE.md limits UI complexity. Any material UI change requires a new planning decision (new architecture decision record), not an inline edit.
**Recommended action:** [STAGE-2-RESOLVED — 2026-06-11] No action required within current PBVI scope. IC-11 enforcement confirmed from M-004 code review — no transformation logic in _HTML JavaScript. Flag explicitly in any future enhancement proposing UI changes; IC-11 has no automated test (see R-007).
**Severity rationale:** P3 — risk is scope-bounded. Current UI scope is minimal and not expected to change within PBVI governance.

---

## R-003 — Docker Volume Bypass of init.sql Seeding

**Risk ID:** R-003
**Description:** The `/docker-entrypoint-initdb.d/` mechanism executes `init.sql` only on first container start, when the data volume is empty. `docker compose down` without `-v` leaves the Postgres data volume; `init.sql` does not re-execute on the next `docker compose up`. Schema exists but seed data may be stale or absent. Affects local development and verification workflows — not a production concern at current scope.
**Severity:** P3
**Source artifact:** docs/architecture.md R-03; DD-04 Challenge verdict
**Affected module:** M-007, M-008
**Invariants at risk:** IC-1
**Status:** Mitigated — documented in README (confirmed in VERIFICATION_RECORD.md T5.3); Session 5 Final Gate mandated `docker compose down -v` before cold-start verification.
**Mitigation (from docs):** (1) `CREATE TABLE IF NOT EXISTS` prevents duplicate-table errors. (2) README documents `docker compose down -v` as correct teardown with explicit warning. (3) Session 5 gate mandates `-v` before cold-start verification run.
**Recommended action:** [STAGE-2-RESOLVED — 2026-06-11] No code change required. Operational: always use `docker compose down -v` for fresh-seed verification. If engineer-error frequency justifies it, a Makefile target wrapping `down -v && up -d` would make the correct teardown the default path.
**Severity rationale:** P3 — documented, mitigated, and process-controlled. No code change needed.

---

## R-004 — Per-Request psycopg2 Connection Under Concurrent Load

**Risk ID:** R-004
**Description:** Each request to `GET /customers/{customer_id}` opens and closes a psycopg2 connection. Under concurrent load, connection establishment overhead compounds. Postgres has a default connection limit (typically 100); if multiple operations staff query simultaneously, connection exhaustion is possible in a high-concurrency scenario.
**Severity:** P3
**Source artifact:** docs/architecture.md R-04; DD-03 Challenge verdict
**Affected module:** M-002, M-005
**Invariants at risk:** IC-2, IC-9
**Status:** Accepted — documented in docs/architecture.md. System is explicitly an internal operations tool; concurrent usage profile does not justify a connection pool. Fixed stack prohibits ORM and mandates psycopg2.
**Mitigation (from docs):** Accepted and documented. Any future production adaptation must replace per-request connections with a connection pool (e.g., psycopg2 with a pool library, or switching to asyncpg).
**Recommended action:** [STAGE-2-RESOLVED — 2026-06-11] No action within current PBVI scope. Note: "too many connections" OperationalError from Postgres is caught by `except Exception:` in M-002 → HTTP 500 with static literal — IC-9 is preserved even in connection exhaustion scenarios. Flag in any future enhancement increasing concurrency profile.
**Severity rationale:** P3 — accepted within stated scope. Would escalate to P2 if concurrent usage profile changes materially.

---

## R-005 — Auth Dependency Omitted from a Future Route

**Risk ID:** R-005
**Description:** Per-route FastAPI dependency injection requires each new route to declare `Depends(verify_api_key)` explicitly. A developer adding a new route without the dependency creates an unauthenticated endpoint that bypasses the access control boundary. This failure mode is invisible until the route is called without credentials.
**Severity:** P3
**Source artifact:** docs/architecture.md R-05; DD-02 Challenge verdict
**Affected module:** M-001, M-006
**Invariants at risk:** IC-7
**Status:** Accepted within PBVI scope — docs/CLAUDE.md scope boundary prohibits Claude Code from adding routes outside the execution plan. Three routes are defined and fixed for this system.
**Mitigation (from docs):** CLAUDE.md scope boundary limits routes to exactly `GET /health`, `GET /`, `GET /customers/{customer_id}`. This risk applies to future development outside PBVI governance.
**Recommended action:** [STAGE-2-RESOLVED — 2026-06-11] No action within current PBVI scope. If the system is extended beyond PBVI governance: replace per-route `Depends(verify_api_key)` with global middleware + allowlist for unauthenticated routes (rejected DD-02 alternative — the safer default for a growing API). Any new route must be reviewed against this risk before merge.
**Severity rationale:** P3 — scoped away within current PBVI governance. Would escalate to P1 in any production extension where new routes are expected.

---

## R-006 — Postgres Port 5432 Exposed on Host

**Risk ID:** R-006
**Description:** `docker-compose.yml` declares `ports: 5432:5432` on the `db` service, binding Postgres to the host network on port 5432. Any process on the host with valid Postgres credentials (POSTGRES_USER, POSTGRES_PASSWORD from `.env`) can connect directly to `customer_risk_profiles` and execute arbitrary SQL — including INSERTs, UPDATEs, DELETEs, and DDL — bypassing all application-layer invariants. The application's entire security model (IC-3 no writes, IC-4 parameterised queries, IC-7 auth before DB, IC-8 key not in response, IC-9 error isolation) is contingent on callers using the HTTP API. The Postgres port breaks this assumption.
**Severity:** P2
**Source artifact:** STAGE-2-DIVERGENCE in MODULE_CONTRACTS.md M-008; INTEGRATION_CONTRACTS.md Gaps field
**Affected module:** M-008
**Invariants at risk:** IC-3, IC-4, IC-7, IC-8, IC-9
**Status:** Open — not previously captured in risk register. Acceptable on a developer workstation with no other users. P2 for any shared-host, staging, or production deployment.
**Mitigation:** For production or shared-host deployments: remove `ports: 5432:5432` from docker-compose.yml (or replace with `expose:`) so Postgres is only reachable on the Docker Compose internal network. No API change required.
**Recommended action:** [STAGE-2-NEW — 2026-06-11] Document explicitly in README that `ports: 5432:5432` is a development convenience only. Any production or shared-host deployment must remove or restrict this port mapping. Removing it is a one-line change to docker-compose.yml with no application code impact.
**Severity rationale:** P2 — not P1 because the system is documented as a developer/operations tool, not a production service with untrusted hosts. Would escalate to P1 for any deployment context with untrusted host access.

---

## R-007 — IC-10, IC-11, and IC-14 Have No Automated Regression Test

**Risk ID:** R-007
**Description:** Three invariants have no automated regression coverage: IC-10 (no external network calls — manual grep only), IC-11 (UI renders without transformation — manual code review + browser devtools), and IC-14 (connection closed per request — code-observed only, no test). A code change violating any of these three invariants would not be caught by M-010, M-011, or M-012 before merge.
- IC-10 violation: adding `import requests` or a CDN link in `_HTML` introduces an external network dependency and breaks operational isolation.
- IC-11 violation: adding a `{ LOW: 'Low Risk' }` label map or a `.filter()` call in the JavaScript `lookup()` function causes the UI to misrepresent API data.
- IC-14 violation: moving or removing `finally: conn.close()` in M-005 causes connection leaks, which under concurrent load exhaust Postgres `max_connections` and produce cascading 500s.
**Severity:** P2
**Source artifact:** INVARIANT_CATALOGUE.md coverage gap; MODULE_CONTRACTS.md M-012 Known Fragility
**Affected modules:** M-004 (IC-11), M-005 (IC-14), M-006 (IC-10)
**Invariants at risk:** IC-10, IC-11, IC-14
**Status:** Open — no automated tests exist for these three invariants. Confirmed from M-012 Known Fragility and INVARIANT_CATALOGUE.md Automated Coverage Summary.
**Mitigation (partial):** Current CLAUDE.md scope boundary reduces the likelihood of IC-10 and IC-11 violations within PBVI governance. IC-14 is structural — the `finally` clause is unlikely to be accidentally removed. Risk is higher for future out-of-PBVI development.
**Recommended action:** [STAGE-2-NEW — 2026-06-11] For IC-10: add a CI step that greps api/main.py and api/requirements.txt for banned external imports (`requests`, `httpx`, `aiohttp`, `boto3`, `urllib.request`) and fails if found. For IC-14: add a test that verifies Postgres connection count does not increase after a series of requests (query `pg_stat_activity`). For IC-11: browser automation test (Playwright or Selenium) that compares fetch response fields to rendered DOM text. Priority: IC-14 test is highest value (connection leak under load is a silent outage mode); IC-10 grep is lowest effort.
**Severity rationale:** P2 — invariants are currently enforced by code; the gap is that regressions are not automatically detected. Not P1 because current enforcement is confirmed and PBVI scope boundary reduces change surface.

---

## R-008 — No Version Pins in requirements.txt

**Risk ID:** R-008
**Description:** `api/requirements.txt` contains unpinned package dependencies. A breaking upstream release of `fastapi`, `psycopg2-binary`, or `uvicorn` produces a broken or behaviour-changed image on the next `docker compose up --build` without any local change. The image is not reproducible across build environments or time.
**Severity:** P2
**Source artifact:** MODULE_CONTRACTS.md M-009 Known Fragility
**Affected module:** M-009
**Invariants at risk:** IC-2, IC-9 (a behaviour-changed psycopg2 or FastAPI could alter status code or error surface behaviour unpredictably)
**Status:** Open — no version pins present. Confirmed from M-009 source file read.
**Mitigation:** None currently — builds depend on PyPI state at build time.
**Recommended action:** [STAGE-2-NEW — 2026-06-11] Pin all packages in requirements.txt to exact versions matching the verified build (the build that passed 34/34 tests in VERIFICATION_RECORD.md). Format: `fastapi==X.Y.Z`, `psycopg2-binary==X.Y.Z`, `uvicorn==X.Y.Z`. Version numbers are NOT DETERMINABLE FROM SOURCE — engineer must retrieve them from the verified environment (e.g., `pip freeze` inside the running container). This is a low-effort, high-value change that makes the build reproducible and prevents silent breakage.
**Severity rationale:** P2 — unpinned dependencies are a latent risk on every build, not just at enhancement time. Not P1 because the current verified build works and breaking changes in these packages are typically flagged with major version bumps.

---

## Risk Summary

| Risk ID | Description (condensed) | Severity | Status | Invariants at risk | Affected modules |
|---|---|---|---|---|---|
| R-001 | psycopg2 exception leaks into HTTP response | P2 | Mitigated + verified | IC-9 | M-002 |
| R-002 | Inline HTML grows beyond maintainability threshold | P3 | Accepted (scope-bounded) | IC-11 | M-004 |
| R-003 | Docker volume bypass of init.sql seeding | P3 | Mitigated (documented) | IC-1 | M-007, M-008 |
| R-004 | Per-request psycopg2 connection under concurrent load | P3 | Accepted | IC-2, IC-9 | M-002, M-005 |
| R-005 | Auth dependency omitted from a future route | P3 | Accepted (scope-bounded) | IC-7 | M-001, M-006 |
| R-006 | Postgres port 5432 exposed on host | P2 | Open (deployment scope) | IC-3, IC-4, IC-7, IC-8, IC-9 | M-008 |
| R-007 | IC-10, IC-11, IC-14 have no automated regression test | P2 | Open | IC-10, IC-11, IC-14 | M-004, M-005, M-006 |
| R-008 | No version pins in requirements.txt | P2 | Open | IC-2, IC-9 | M-009 |

Stage 2 additions: R-006 (Postgres port exposure — M-008 STAGE-2-DIVERGENCE), R-007 (automated coverage gaps — INVARIANT_CATALOGUE coverage summary), R-008 (no version pins — M-009 Known Fragility).