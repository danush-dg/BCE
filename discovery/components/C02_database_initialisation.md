## C02 — database initialisation
ID: M-007
Layer: infra
Source file: db/init.sql

---

**Module** — database initialisation
**ID** — M-007
**Layer** — infra
**Primary Responsibility** — Define the `customer_risk_profiles` table schema and insert the authoritative seed data. Executed once by the Postgres Docker image on first container start via the `/docker-entrypoint-initdb.d/` mechanism. Not called at application runtime.

**Inputs**
- Executed by Postgres 15 at container initialisation. No inputs from the application layer.
- `env_file: .env` provides `POSTGRES_USER` and `POSTGRES_DB` (used by the container's entrypoint for superuser setup, not by init.sql directly).

**Outputs**
`customer_risk_profiles` table — confirmed schema from source:

| Column | Type | Constraints |
|---|---|---|
| `customer_id` | `VARCHAR` | `PRIMARY KEY` |
| `risk_tier` | `VARCHAR` | `NOT NULL`, `CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH'))` |
| `risk_factors` | `TEXT[]` | `NOT NULL` |

**Seed data** — 9 rows, 3 per tier:

| customer_id | risk_tier | risk_factors (count) |
|---|---|---|
| CUST-001 | LOW | 3 factors |
| CUST-002 | LOW | 2 factors |
| CUST-003 | LOW | 3 factors |
| CUST-004 | MEDIUM | 2 factors |
| CUST-005 | MEDIUM | 2 factors |
| CUST-006 | MEDIUM | 2 factors |
| CUST-007 | HIGH | 2 factors |
| CUST-008 | HIGH | 2 factors |
| CUST-009 | HIGH | 3 factors |

`risk_factors` values are plain English descriptive strings (e.g., `'account in good standing'`, `'three missed payments'`).

**Public Interface**
- Not callable at runtime. Static SQL file executed once at DB container initialisation.
- `CREATE TABLE IF NOT EXISTS` — idempotent on re-run if volume is not cleared. INSERT statements are not idempotent — will raise duplicate primary key errors if run a second time on an existing table. This is harmless in practice because Postgres's `/docker-entrypoint-initdb.d/` mechanism only executes on an empty data directory.

**Error Behaviour**
- Schema creation failure: STARTUP-FATAL for the db container — api container never starts.
- `CREATE TABLE IF NOT EXISTS`: prevents error if table already exists (volume not cleared between runs).
- INSERT duplicate key: would fail if init.sql somehow re-executed against an existing table. Not expected in normal operation.

**Known Fragility**
- `docker compose down` **without** `-v` leaves the Postgres data volume intact. On next `docker compose up`, init.sql does not re-execute (data directory is not empty). Seed data persists from the prior run. If seed data was manually modified, the discrepancy is silent (R-003).
- `risk_tier VARCHAR NOT NULL CHECK (...)` — the CHECK constraint enforces INV-06 at the data layer. If this constraint were weakened or removed, invalid tier values could be inserted and returned by the API without any application-level violation.
- `risk_factors TEXT[] NOT NULL` — NOT NULL means every row must have at least an empty array. psycopg2 maps `TEXT[]` to a Python list; an empty array maps to `[]`. INV-05 (risk_factors must be an array) is enforced at both DB level (type) and psycopg2 mapping level.
- No schema migration system — adding or modifying columns requires manual `ALTER TABLE` or re-seeding with a cleared volume.

**Change Impact**
- Schema changes (adding a column, renaming a column) require corresponding changes to M-005's SELECT query (INV-04), response dict (INV-05), and test assertions in M-010/M-012.
- Removing the `CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH'))` constraint removes the data-layer enforcement of INV-06 — only application-level verification would remain.
- Modifying seed data affects test fixtures in M-010, M-011, and M-012 (tests reference CUST-001, CUST-004, CUST-007 by ID and expect specific tier values).

**Callers** — Postgres 15 Docker image entrypoint at first container start (not the application)
**Calls** — None
**Integration Points Used** — None
