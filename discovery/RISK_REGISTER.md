STAGE-1-DRAFT: DOCS-DERIVED — 2026-06-11 — Produced by BCE Adapter Pipeline Stage 1

# RISK_REGISTER.md — Customer Risk API
Produced by: BCE Adapter Pipeline Stage 1 (CC)
Sources: docs/architecture.md (Sections 3–4), docs/CLAUDE.md, docs/VERIFICATION_RECORD.md

Note: No docs/PHASE4_GATE_RECORD.md present. All risks sourced from docs/architecture.md Section 4
(Key Risks) and Section 3 (Challenge My Decisions — challenge verdict rows). Severity mapped from
architecture risk language to BCE P1/P2/P3 scale. Stage 2 will add code-observed risks and recommended
actions NOT DETERMINABLE FROM SOURCE at this stage.

---

## R-001 — psycopg2 Exception Surfaces in HTTP Response

**Risk ID:** R-001
**Description:** If a database connection fails or query raises and is not caught (or is caught too narrowly), the raw psycopg2 traceback reaches the 500 response body, exposing table names, column names, psycopg2 connection strings, and application file paths.
**Severity:** P2
**Source artifact:** docs/architecture.md R-01
**Invariants at risk:** INV-09 — only pre-defined static strings permitted in error responses
**Status:** Mitigated — verified in VERIFICATION_RECORD.md (T2.3 and T3.3): HTTPException(500, "Internal server error") confirmed; psycopg2 detail confirmed absent from response body. Verification test TestInv09InternalErrorIsolation passes from cold start.
**Mitigation (from docs):** `get_db_connection` must wrap all psycopg2 exceptions and return a sanitised 500 with a static error message. Session 3 (T3.3) verified by stopping the db container and confirming response body.
**Recommended action:** NOT DETERMINABLE FROM SOURCE — Stage 2 code review of `get_db_connection` will confirm exception handler scope (should be `except Exception` or `except psycopg2.Error` — narrower types risk uncaught exceptions slipping through). Ongoing: this risk is active for any future changes to `get_db_connection`.
**Severity rationale:** Downgraded from P1 to P2 because mitigated and verified. Remains P2 (not P3) because this is the highest-consequence failure mode in the system — any regression here immediately violates INV-09 in a security-impactful way.

---

## R-002 — Inline HTML Grows Beyond Maintainability Threshold

**Risk ID:** R-002
**Description:** Embedding HTML and JavaScript in Python source (main.py) couples UI changes to API container rebuilds. As UI complexity grows, the inline string becomes difficult to read, modify, and test safely. An error in the inline string may produce a silent runtime failure (malformed HTML served) rather than a build-time error.
**Severity:** P3
**Source artifact:** docs/architecture.md R-02; DD-01 Challenge verdict
**Invariants at risk:** INV-11 — UI fidelity. Complexity increases the risk that a developer inadvertently introduces a tier remapping or conditional display transformation.
**Status:** Accepted — scope boundary in docs/CLAUDE.md limits UI to one text input and one result display. No additional states or interactivity permitted within PBVI scope.
**Mitigation (from docs):** Scope boundary in CLAUDE.md limits UI complexity. Any material UI change requires a new planning decision (new architecture decision record), not an inline edit.
**Recommended action:** NOT DETERMINABLE FROM SOURCE — no action required within current PBVI scope. Document this risk explicitly in any future enhancement that proposes UI changes.
**Severity rationale:** P3 — risk is scope-bounded. Current UI scope is minimal and not expected to change within PBVI governance.

---

## R-003 — Docker Volume Bypass of init.sql Seeding

**Risk ID:** R-003
**Description:** The `/docker-entrypoint-initdb.d/` mechanism executes `init.sql` only on first container start, when the data volume is empty. `docker compose down` without `-v` leaves the Postgres data volume; `init.sql` does not re-execute on the next `docker compose up`. Schema exists but seed data may be stale or absent. Affects local development and verification workflows — not a production concern at current scope.
**Severity:** P3
**Source artifact:** docs/architecture.md R-03; DD-04 Challenge verdict
**Invariants at risk:** INV-01 — if seed data is stale, the DB-to-API comparison in T3.2 may compare against outdated ground truth.
**Status:** Mitigated — documented in README (confirmed in VERIFICATION_RECORD.md T5.3); Session 5 Final Gate mandated `docker compose down -v` before cold-start verification.
**Mitigation (from docs):** (1) `CREATE TABLE IF NOT EXISTS` prevents duplicate-table errors. (2) README documents `docker compose down -v` as correct teardown with explicit warning. (3) Session 5 gate mandates `-v` before cold-start verification run.
**Recommended action:** NOT DETERMINABLE FROM SOURCE — no code change required. Operational reminder: always use `-v` in teardown for fresh-seed verification. Consider adding a comment or validation step in a future Makefile or helper script if this becomes a recurring developer error.
**Severity rationale:** P3 — documented, mitigated, and process-controlled. No code change needed.

---

## R-004 — Per-Request psycopg2 Connection Under Concurrent Load

**Risk ID:** R-004
**Description:** Each request to `GET /customers/{customer_id}` opens and closes a psycopg2 connection. Under concurrent load, connection establishment overhead compounds. Postgres has a default connection limit (typically 100); if multiple operations staff query simultaneously, connection exhaustion is possible in a high-concurrency scenario.
**Severity:** P3
**Source artifact:** docs/architecture.md R-04; DD-03 Challenge verdict
**Invariants at risk:** INV-02 — connection exhaustion under load could produce unexpected status codes (500 from connection failure rather than 200 or 404). INV-09 — uncaught Postgres "too many connections" exception would need to be handled to avoid detail leak.
**Status:** Accepted — documented in docs/architecture.md. System is explicitly an internal operations tool; concurrent usage profile does not justify a connection pool. Fixed stack prohibits ORM and mandates psycopg2.
**Mitigation (from docs):** Accepted and documented. Any future production adaptation must replace per-request connections with a connection pool (e.g., psycopg2 with a pool library, or switching to asyncpg).
**Recommended action:** NOT DETERMINABLE FROM SOURCE — no action within current PBVI scope. Flag explicitly in any future enhancement that increases expected concurrency or proposes production hardening.
**Severity rationale:** P3 — accepted within stated scope. Would escalate to P2 if concurrent usage profile changes materially.

---

## R-005 — Auth Dependency Omitted from a Future Route

**Risk ID:** R-005
**Description:** Per-route FastAPI dependency injection requires each new route to declare `Depends(verify_api_key)` explicitly. A developer adding a new route without the dependency creates an unauthenticated endpoint that bypasses the access control boundary. This failure mode is invisible until the route is called without credentials.
**Severity:** P3
**Source artifact:** docs/architecture.md R-05; DD-02 Challenge verdict
**Invariants at risk:** INV-07 — authentication gate. A new unauthenticated route would expose customer data or internal state to any caller.
**Status:** Accepted within PBVI scope — docs/CLAUDE.md scope boundary prohibits Claude Code from adding routes outside the execution plan. Three routes are defined and fixed for this system.
**Mitigation (from docs):** CLAUDE.md scope boundary limits routes to exactly `GET /health`, `GET /`, `GET /customers/{customer_id}`. This risk applies to future development outside PBVI governance.
**Recommended action:** NOT DETERMINABLE FROM SOURCE — no action within current PBVI scope. If the system is extended beyond PBVI governance, consider global middleware with an allowlist for unauthenticated routes (the rejected DD-02 alternative) as the safer default for a growing API.
**Severity rationale:** P3 — scoped away within current PBVI governance. Would escalate to P1 in any production extension where new routes are expected.

---

## Risk Summary

| Risk ID | Description (condensed) | Severity | Status | Invariants at risk |
|---|---|---|---|---|
| R-001 | psycopg2 exception leaks into HTTP response | P2 | Mitigated + verified | INV-09 |
| R-002 | Inline HTML grows beyond maintainability threshold | P3 | Accepted (scope-bounded) | INV-11 |
| R-003 | Docker volume bypass of init.sql seeding | P3 | Mitigated (documented) | INV-01 |
| R-004 | Per-request psycopg2 connection under concurrent load | P3 | Accepted | INV-02, INV-09 |
| R-005 | Auth dependency omitted from a future route | P3 | Accepted (scope-bounded) | INV-07 |

No PHASE4_GATE_RECORD.md present — no additional ACCEPT decisions to incorporate. Stage 2 will extend with risks surfaced by code extraction (e.g., observed exception handler scope, actual SQL query construction pattern, discovered fragilities not visible from docs).
