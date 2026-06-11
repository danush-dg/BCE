## C03 — container orchestration
ID: M-008
Layer: infra
Source file: docker-compose.yml

---

**Module** — container orchestration
**ID** — M-008
**Layer** — infra
**Primary Responsibility** — Declare and configure the two-container stack (`db` and `api`), health-based startup sequencing, port mappings, and environment variable injection. The single source of truth for how the system is assembled and started.

**Inputs**
- `.env` file injected via `env_file: .env` on both services: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_HOST`, `POSTGRES_PORT`, `API_KEY`.
- `db/init.sql` — mounted as a volume into `/docker-entrypoint-initdb.d/init.sql` on the `db` service.
- `./api/` directory — built as the container image for the `api` service via `build: ./api`.

**Outputs**
**`db` service:**
- `image: postgres:15` — Postgres 15 container
- Port `5432:5432` — Postgres accessible on host port 5432 (see Known Fragility)
- `healthcheck: test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"], interval: 5s, retries: 5`
- `$$POSTGRES_USER` and `$$POSTGRES_DB` — double `$` escapes to single `$` at runtime (Compose YAML escaping rule)

**`api` service:**
- `build: ./api` — image built from api/Dockerfile (M-009)
- Port `8000:8000` — API accessible on host port 8000
- `depends_on: db: condition: service_healthy` — api container blocked until db passes `pg_isready` healthcheck

**Public Interface**
- `docker compose up -d` — full stack start (detached)
- `docker compose down` — stops containers, preserves data volume
- `docker compose down -v` — stops containers, destroys data volume (required for clean seed re-run)
- `docker compose stop db` / `docker compose start db` — used by M-012 (TestInv09) to simulate DB failure

**Error Behaviour**
- DB healthcheck fails all 5 retries (25s): api service never starts — STARTUP-FATAL.
- DB container fails to start: api service never starts — STARTUP-FATAL.
- No `start_period` configured for healthcheck — retries begin immediately on container start. Postgres may not be ready within the first retry; this is absorbed by the 5-retry budget.
- No `timeout` specified in healthcheck — Docker default is 30s per probe. `pg_isready` completes in milliseconds on a healthy instance; the 30s timeout is effectively never reached.

**Known Fragility**
- **Port 5432:5432 exposed on `db` service**: Postgres is reachable on host port 5432. Any process on the host machine with valid credentials can connect directly to the database, bypassing the API layer, its auth gate, and all application-level invariants (INV-07, INV-03, INV-04). This is a development-scope configuration — acceptable for the stated internal-ops use case (R-003 adjacent), but a P2-level risk for any production deployment or shared development host. [STAGE-2-DIVERGENCE — 2026-06-11]: Stage 1 noted "no external networks defined" as the isolation evidence, but did not flag port 5432 exposure. Code shows db port 5432 is mapped to host — direct DB access is possible without API auth. [RESOLVED — 2026-06-11]: CONFIRMED. Stage 1 was correct but incomplete — no external Docker networks is true, but host port mapping creates a direct-access side-channel not addressed in Stage 1. Finding stands. Captured in RISK_REGISTER.md as R-006 (P2). Accepted for development scope; action required before any shared-host or production deployment.
- No external network blocks defined — confirmed INV-10 at the Compose layer. Neither service declares `networks:` with `external: true`.
- M-012 (test_invariants.py TestInv09) calls `docker compose stop db` / `docker compose start db` from test code. This is a legitimate test technique but means test execution can mutate the running stack state. If tearDownClass fails (e.g., test runner killed mid-test), the `db` container remains stopped and subsequent tests fail with unexpected errors.

**Change Impact**
- Removing `condition: service_healthy` from `depends_on` risks the api container starting before Postgres is ready — connection errors at cold start producing unexpected 500s (INV-02 violation window).
- Adding `external: true` to a network definition violates INV-10.
- Removing port `5432:5432` from the `db` service would close the direct DB access path — desirable for hardening, required by M-010/M-011/M-012 tests that use `docker compose exec db psql` (which uses Docker exec, not host TCP — would still work without host port mapping).
- Adding a third service (e.g., Redis cache) would require a new `depends_on` entry on `api` and would violate INV-01 if the cache were used for response caching.

**Callers** — Docker Compose CLI (engineer, CI)
**Calls** — None
**Integration Points Used** — None
