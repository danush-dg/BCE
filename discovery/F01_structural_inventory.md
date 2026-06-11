STAGE-F1-DRAFT: STRUCTURAL
Produced by: BCE Stage 2 Session F01 (CC)
Date: 2026-06-11

Canonical layer boundary: customer_risk_profiles table in riskdb Postgres database
Source files read: db/init.sql
Disposition: FULL EXTRACTION

Pre-condition assessment:
- Step 1 (Catalog check): No maintained metadata catalog present. No AWS Glue, Unity Catalog,
  Collibra, Alation, Apache Atlas, or equivalent referenced in INTAKE_SUMMARY.md or codebase.
- Step 2 (Structural check): db/init.sql contains CREATE TABLE DDL → FULL EXTRACTION.
- Step 3 (Canonical boundary): customer_risk_profiles table in riskdb Postgres database.
  Extraction scoped to this table only. No other tables exist in init.sql.

Source priority order applied:
1. ORM model definitions — NOT PRESENT
2. Migration scripts — NOT PRESENT
3. Schema dump / DDL — PRESENT: db/init.sql (sole source; no drift possible)
4. GraphQL schema — NOT PRESENT
5. OpenAPI / JSON Schema — NOT PRESENT
6. ERD / data dictionary — NOT PRESENT

---

## Entity Inventory

| Entity/Table Name | Column | Type | Constraints | Notes |
|---|---|---|---|---|
| customer_risk_profiles | customer_id | VARCHAR | PRIMARY KEY | Natural key — business-assigned. Format CUST-NNN observed in seed data. |
| customer_risk_profiles | risk_tier | VARCHAR | NOT NULL, CHECK | Controlled vocabulary: CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH')). |
| customer_risk_profiles | risk_factors | TEXT[] | NOT NULL | Postgres native array type. Plain-English factor strings. NOT NULL does not prevent empty array. |

Full DDL (db/init.sql lines 1–6):
```sql
CREATE TABLE IF NOT EXISTS customer_risk_profiles (
    customer_id  VARCHAR PRIMARY KEY,
    risk_tier    VARCHAR NOT NULL,
    risk_factors TEXT[]  NOT NULL,
    CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH'))
);
```

Notes:
- Single-table schema. No other tables exist in db/init.sql or within the canonical boundary.
- VARCHAR columns have no length limit — Postgres treats unlimited VARCHAR identically to TEXT.
- CREATE TABLE IF NOT EXISTS — DDL is idempotent; seed INSERTs are not guarded against duplication.
- All 9 seed rows have non-empty risk_factors arrays (1–3 factors each).

---

## Relationship Inventory

| Relationship | Declaration Type | Source Entity | Target Entity | Notes |
|---|---|---|---|---|
| — | — | — | — | No relationships. Single-table schema. No foreign keys, no join tables, no ORM associations. |

---

## Constraint Inventory

| Entity | Constraint Type | Fields | Notes |
|---|---|---|---|
| customer_risk_profiles | UNIQUE (PRIMARY KEY) | customer_id | Implicit unique B-tree index. No two rows may share a customer_id. |
| customer_risk_profiles | NOT_NULL | risk_tier | Explicit NOT NULL. Combined with CHECK — no null and no out-of-vocabulary tier. |
| customer_risk_profiles | NOT_NULL | risk_factors | Explicit NOT NULL. Empty array {} is still permitted. |
| customer_risk_profiles | CHECK | risk_tier | CHECK (risk_tier IN ('LOW', 'MEDIUM', 'HIGH')) — unnamed table-level constraint. Enforces IC-6 at the data layer. Applied on INSERT and UPDATE. |

Summary: 4 constraints across 3 fields — 1 PK/UNIQUE, 2 NOT NULL, 1 CHECK.
No UNIQUE constraints beyond PK. No INDEX declarations. No triggers, sequences, views, or functions.

---

## Divergence Flags

| Flag ID | Source A | Source B | What Diverges | Notes |
|---|---|---|---|---|
| — | — | — | — | No divergences. Single schema source (db/init.sql). No ORM, migration history, or schema dump to compare against. |

---

## Seed Data Summary (db/init.sql lines 8–17)

9 rows — 3 per tier. Handed to F02 for vocabulary extraction.

| customer_id | risk_tier | risk_factors count |
|---|---|---|
| CUST-001 | LOW | 3 |
| CUST-002 | LOW | 2 |
| CUST-003 | LOW | 3 |
| CUST-004 | MEDIUM | 2 |
| CUST-005 | MEDIUM | 2 |
| CUST-006 | MEDIUM | 2 |
| CUST-007 | HIGH | 2 |
| CUST-008 | HIGH | 2 |
| CUST-009 | HIGH | 3 |

- risk_tier observed values: LOW, MEDIUM, HIGH — exactly matches CHECK constraint. All uppercase.
- risk_factors cardinality: 2–3 per row. No empty arrays in seed data.
- customer_id pattern: CUST-NNN format (3-digit zero-padded integer suffix). Seed covers CUST-001 through CUST-009.

---

F01 is complete. discovery/F01_structural_inventory.md must be committed and reviewed before F02 begins.
