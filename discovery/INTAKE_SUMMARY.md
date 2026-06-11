STAGE-1-DRAFT: DOCS-DERIVED — 2026-06-11 — Produced by BCE Adapter Pipeline Stage 1

# INTAKE_SUMMARY.md — Customer Risk API
**Date:** 2026-06-11
**Engineer:** DataGrokr Engineering
**Path:** C — PBVI Adapter
**Canonical layer boundary:** `customer_risk_profiles` table in `riskdb` Postgres database — defined and seeded in db/init.sql. Single-table schema; no additional layers or schemas.
**Session F disposition:** FULL EXTRACTION — db/init.sql contains CREATE TABLE DDL with one entity (customer_risk_profiles). No maintained metadata catalog detected. Canonical layer boundary declared above.

---

## System Purpose

The Customer Risk API provides authenticated, read-only HTTP access to pre-assessed customer risk tier data stored in a Postgres database. It replaces ad-hoc SQL queries by operations staff with a single defined interface: one endpoint, one API key, one response shape. The system surfaces data — it does not compute, transform, or assess risk. Risk tier data is pre-populated in the database; the API's role is a controlled access boundary over that data, ensuring database credentials are never distributed to operations staff and that schema and row-level data remain hidden behind a defined contract.

---

## Known Architecture

**Route structure (three routes exactly):**
- `GET /health` — unauthenticated; returns `{"status": "ok"}` only
- `GET /` — unauthenticated; returns inline HTML + JavaScript browser UI
- `GET /customers/{customer_id}` — requires `X-API-Key` header; returns customer risk record or 404

**Five key design decisions (from docs/architecture.md):**

DD-01 — Inline HTML UI: The browser UI is returned as an `HTMLResponse` from `GET /`. HTML and embedded JavaScript live in Python source (main.py). No `StaticFiles` mount; no separate file deployment artefact. Rationale: eliminates path misconfiguration as a deployment failure mode.

DD-02 — Route-level auth dependency: `verify_api_key` is declared as a FastAPI `Depends()` dependency on the customer lookup route only. Auth is a structural gate on the route — not global middleware. `/health` and `/` are intentionally unauthenticated.

DD-03 — Per-request psycopg2 connection: Each authenticated request opens a psycopg2 connection, executes a single parameterised SELECT, and closes the connection in a finally block. No connection pool. No ORM. System is an internal operations tool; per-request connections are acceptable at this concurrency profile.

DD-04 — Postgres initialised via init.sql: `db/init.sql` contains `CREATE TABLE IF NOT EXISTS` and all `INSERT` rows. Postgres Docker image executes it via `/docker-entrypoint-initdb.d/` on first container start. No migration tooling; no application-level schema initialisation.

DD-05 — Same port, same process: `GET /` returns HTML; `GET /customers/{customer_id}` returns JSON. Both served from the same FastAPI process on port 8000. JavaScript makes same-origin fetch calls — no CORS configuration required.

**Fixed stack:** Docker Compose · Postgres 15 · FastAPI · Python 3.11 · psycopg2-binary · Inline HTML + Vanilla JavaScript. No additions permitted.

**Database schema:** One table — `customer_risk_profiles`:
- `customer_id` VARCHAR PRIMARY KEY
- `risk_tier` VARCHAR NOT NULL CHECK (IN ('LOW', 'MEDIUM', 'HIGH'))
- `risk_factors` TEXT[]
- Seeded with 9 rows minimum, at least 3 per tier, CUST-NNN format IDs

---

## Known Pain Points

Risks documented in docs/architecture.md Section 4. No docs/PHASE4_GATE_RECORD.md present — known pain points sourced from architecture risk register only.

**R-01 — psycopg2 exception leaks into HTTP response (High):** If a database connection fails and is not caught, raw traceback reaches the 500 response body, exposing table names, column names, and connection details. Design intent: `get_db_connection` must catch all psycopg2 exceptions and sanitise to `HTTPException(500, "Internal server error")`. Verified in Session 3 (T3.3) — confirmed working in VERIFICATION_RECORD.md.

**R-02 — Inline HTML grows beyond maintainability threshold (Low):** Embedding HTML in Python source couples UI changes to API container rebuilds. Acceptable at current minimal UI scope (one input, one result display). Scope boundary in CLAUDE.md limits UI complexity — any material UI change requires new planning decision.

**R-03 — Docker volume bypass of init.sql seeding (Low):** `docker compose down` without `-v` leaves the Postgres data volume; `init.sql` does not re-execute on next `up`. `CREATE TABLE IF NOT EXISTS` prevents duplicate-table errors but seed data may be stale. Mitigation: README documents `docker compose down -v` as correct teardown; Session 5 gate mandated `-v` before cold-start verification.

**R-04 — Per-request psycopg2 connection under concurrent load (Low):** Connection overhead compounds under concurrent usage. Accepted and documented — system is an internal operations tool, not a high-throughput service. Any future production adaptation must replace per-request connections with a pool.

**R-05 — Auth dependency omitted from a future route (Low):** Per-route dependency requires each new route to declare auth explicitly. A route added without the dependency creates an unauthenticated endpoint. Mitigated by CLAUDE.md scope boundary prohibiting new routes within PBVI scope.

---

## Documents Reviewed

| Document | One-line contribution |
|---|---|
| docs/architecture.md | Problem framing, five key design decisions (DD-01–DD-05), key risks (R-01–R-05), key assumptions (A-01–A-06), open questions (OQ-01–OQ-04) — primary architecture source |
| docs/invariant.md | 13 invariants (INV-01–INV-13) covering data correctness, authentication, response shape, error surface isolation, and UI fidelity — primary source for INVARIANT_CATALOGUE.md |
| docs/execution_plan.md | Five-session build plan (S1–S5), permitted error literals, task-level inputs/outputs, and invariant touch map — primary source for module contract skeletons |
| docs/CLAUDE.md | Frozen execution contract: hard invariants, scope boundary, fixed stack, permitted file list, prohibited operations — confirms three-route architecture |
| docs/VERIFICATION_RECORD.md | Session-by-session verification evidence — confirms system fully built and tested (34/34 tests passing from cold start); confirms OQ-01 through OQ-03 resolved |
| PROJECT_MANIFEST.md | NOT PRESENT — no manifest file in repository root |
| docs/PHASE4_GATE_RECORD.md | NOT PRESENT — risk register derived from docs/architecture.md directly |

---

## Open Questions Before Extraction

From docs/architecture.md Section 6. Resolution status assessed from docs/VERIFICATION_RECORD.md and docs/execution_plan.md evidence.

| ID | Question | Status |
|---|---|---|
| OQ-01 | risk_factors shape: plain strings or objects with name/description? Can list be empty? | Resolved — TEXT[] of plain strings; seed data contains non-empty arrays (confirmed in T1.2 and T3.2 of VERIFICATION_RECORD.md) |
| OQ-02 | 404 response shape: does it match the 200 envelope? | Resolved — `{"detail": "Customer not found"}` (FastAPI HTTPException format; confirmed in EXECUTION_PLAN.md permitted literals and VERIFICATION_RECORD.md T3.1) |
| OQ-03 | Customer ID format: string, integer, or UUID? | Resolved — VARCHAR string in CUST-NNN format (confirmed in T1.2 seed data requirements and T3.2 DB output) |
| OQ-04 | API key format and minimum entropy: not specified in brief | Partially resolved — API_KEY is a plain string from .env; no minimum entropy enforced at the application layer. Remains an open operational consideration for production hardening. |

---

## Confidence Assessment

**HIGH** — PBVI artifacts are structured, high-fidelity, and complete. docs/CLAUDE.md is a frozen execution contract with explicit scope boundaries. docs/VERIFICATION_RECORD.md confirms a fully built, tested, and verified system (34/34 tests passing from cold start across INV-01 through INV-13). The system is simple and well-bounded: one Postgres table, three FastAPI routes, no external network integrations outside the Compose stack.
