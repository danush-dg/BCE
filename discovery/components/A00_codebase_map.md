## A00 — Codebase Map
Produced by: BCE Stage 2 Session A0 (CC)
Date: 2026-06-11
Anchored to: discovery/INTAKE_SUMMARY.md

Excluded from traversal: v/ (Python virtual environment — equivalent to node_modules/),
api/__pycache__/ (compiled Python bytecode — generated artifacts), discovery/ (BCE Stage 1
output artifacts, not source code).

---

### [root]
- .env — Runtime secrets file (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB, POSTGRES_HOST, POSTGRES_PORT, API_KEY); populated from .env.example; not committed to version control; contains live credentials
- .env.example — Credential template with the six required environment variable keys and safe defaults (POSTGRES_HOST=db, POSTGRES_PORT=5432); values intentionally blank
- docker-compose.yml — Two-service Compose stack: `db` (postgres:15 with pg_isready healthcheck and init.sql mount) and `api` (builds ./api, port 8000, depends_on db with condition: service_healthy); no external networks defined
- README.md — Operator-facing documentation: what the system is, prerequisites, setup commands, curl usage examples, teardown with -v flag warning, and stack table

### api/
- Dockerfile — Container image definition: FROM python:3.11-slim, installs requirements.txt, copies application source, EXPOSE 8000, CMD uvicorn main:app on 0.0.0.0:8000
- main.py — Complete FastAPI application: _HTML inline UI string constant, FastAPI app instance, verify_api_key async dependency, get_db_connection utility, and three route handlers (GET /health, GET /, GET /customers/{customer_id})
- requirements.txt — Python package dependencies: fastapi, uvicorn, psycopg2-binary, pytest, requests (no version pins)
- test_errors.py — Automated error state tests (5 cases): no key → 401, wrong key → 401, empty key → 401, CUST-999 → 404, both 401 bodies identical; reads API_KEY and BASE_URL from environment
- test_injection.py — SQL injection tests (6 cases): 5 injection payload patterns each returning 404, plus row count unchanged assertion; uses docker compose exec to query Postgres directly
- test_invariants.py — Full invariant verification script (13 test classes, INV-01 through INV-13): includes DB-comparison tests via psql, DB-stop/start lifecycle for INV-09, and row count checks; INV-10 and INV-11 documented as manual checks

### db/
- init.sql — Postgres schema and seed data: CREATE TABLE IF NOT EXISTS customer_risk_profiles (customer_id VARCHAR PK, risk_tier VARCHAR NOT NULL with CHECK constraint, risk_factors TEXT[] NOT NULL); 9 INSERT rows, 3 per tier (CUST-001 through CUST-009)

### docs/
- CLAUDE.md — Frozen execution contract: hard invariants (INV-01 through INV-13), scope boundary (permitted and prohibited file and operation types), fixed stack, and commit message template
- SESSION_LOG.md — Engineer session journal: per-task build narrative for all five sessions (S1–S5), task outcomes, commit hashes, and integration check results
- SKILL.md — BCE core skill file (v2.3) loaded into this project's docs/ directory; same content as docs/bce_core.md; provides BCE methodology reference and extraction session prompts for this project
- VERIFICATION_RECORD.md — Session-by-session verification evidence log: actual command outputs, expected vs. actual comparisons, per-task PASS/FAIL verdicts, and the Session 5 final gate (34/34 tests passing from cold start)
- architecture.md — Architecture decision record: problem framing, five key design decisions (DD-01 through DD-05) with rationale and rejected alternatives, challenge verdicts, key risks (R-01–R-05), key assumptions (A-01–A-06), and open questions (OQ-01–OQ-04)
- bce_core.md — BCE core skill canonical source (v2.3): full BCE methodology including extraction paths, intake step, Stage 1/2/3 execution prompts, Session F Domain Model spec, artifact schemas, and annotation loop procedures
- bce_signal.md — BCE signal pipeline skill: non-code source extraction procedures (Documents, Teams, Git adapters), Stage 1–3 prompts for PATH B extraction, and SIGNAL_GAPS.md structure
- execution_plan.md — Five-session build plan (S1–S5): per-task Claude Code prompts, test cases, verification commands, expected outputs, invariant touch map, and permitted error literals table
- invariant.md — Invariants specification: 13 named invariants (INV-01 through INV-13) with statement, violation consequence, and verification note for each

---

## Cross-reference against INTAKE_SUMMARY.md

INTAKE_SUMMARY.md described the following expected components. Confirmation against traversal:

| Component described in INTAKE_SUMMARY | Present? | File |
|---|---|---|
| FastAPI app with three routes | YES | api/main.py |
| psycopg2 database connection function | YES | api/main.py (get_db_connection) |
| API key auth dependency | YES | api/main.py (verify_api_key) |
| Inline HTML UI | YES | api/main.py (_HTML constant) |
| Postgres schema and seed data | YES | db/init.sql |
| Docker Compose two-service stack | YES | docker-compose.yml |
| Environment variable template | YES | .env.example |
| README | YES | README.md |
| Container image definition | YES | api/Dockerfile |
| Python dependencies | YES | api/requirements.txt |

Additional files present in source, not explicitly described in INTAKE_SUMMARY (but referenced in docs/execution_plan.md):

| File | Notes |
|---|---|
| api/test_errors.py | Error state tests — described in EXECUTION_PLAN.md S4.T4.2; not listed as a main module in INTAKE_SUMMARY |
| api/test_injection.py | SQL injection tests — described in EXECUTION_PLAN.md S5.T5.1 |
| api/test_invariants.py | Full invariant script — described in EXECUTION_PLAN.md S5.T5.2 |
| .env | Live secrets file — expected to exist; not a source artifact |
| docs/SESSION_LOG.md | Engineer session journal — planning/process artifact |
| docs/VERIFICATION_RECORD.md | Verification evidence — planning/process artifact |
| docs/SKILL.md | BCE skill (duplicate of bce_core.md) — process/methodology artifact |
| docs/bce_core.md | BCE skill canonical source — process/methodology artifact |
| docs/bce_signal.md | BCE signal skill — process/methodology artifact |

No unexpected source files. File inventory is consistent with INTAKE_SUMMARY.md system description.

---

A00 is complete. Engineer must review this map against discovery/INTAKE_SUMMARY.md before Sessions A–E begin. Do not proceed to Session A without confirming engineer review.
