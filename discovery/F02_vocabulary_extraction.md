STAGE-F2-DRAFT: VOCABULARY
Produced by: BCE Stage 2 Session F02 (CC)
Date: 2026-06-11

Prerequisite check: F01 carries STAGE-F1-DRAFT: STRUCTURAL (not SPARSE) — full extraction proceeds.

Sources read:
- db/init.sql (seed data — 9 rows, primary vocabulary source)
- api/test_invariants.py (test fixtures — customer ID sentinels, VALID_TIERS set)
- api/test_errors.py (test fixtures — customer ID sentinels, error detail literals)

---

## Per-Field Vocabulary

| Entity | Field | Observed Values | Source |
|---|---|---|---|
| customer_risk_profiles | risk_tier | `LOW` | db/init.sql seed rows CUST-001, CUST-002, CUST-003 |
| customer_risk_profiles | risk_tier | `MEDIUM` | db/init.sql seed rows CUST-004, CUST-005, CUST-006 |
| customer_risk_profiles | risk_tier | `HIGH` | db/init.sql seed rows CUST-007, CUST-008, CUST-009 |
| customer_risk_profiles | risk_tier | `LOW`, `MEDIUM`, `HIGH` (full set declared) | db/init.sql CHECK constraint; test_invariants.py `VALID_TIERS = {"LOW", "MEDIUM", "HIGH"}` |
| customer_risk_profiles | customer_id | `CUST-001` through `CUST-009` | db/init.sql seed data |
| customer_risk_profiles | customer_id | `CUST-999` | test_invariants.py TestInv02, TestInv03; test_errors.py — used as non-existent sentinel (produces 404) |
| customer_risk_profiles | customer_id | `probe-key-that-must-not-appear` | test_invariants.py TestInv08 — used as a fake API key value in the X-API-Key header, NOT as a customer_id |
| customer_risk_profiles | risk_factors | See Risk Factor Inventory below | db/init.sql seed data |

Notes on risk_tier vocabulary:
- Three-value closed set: LOW, MEDIUM, HIGH. All uppercase. No lowercase or mixed-case variants observed anywhere.
- Vocabulary is doubly enforced: at the data layer (CHECK constraint in DDL) and in test code (VALID_TIERS set in TestInv06).
- No ordering metadata in the schema — LOW < MEDIUM < HIGH ordering is implied by naming but not declared.
- No null values in seed data. NOT NULL constraint prevents null at the DB level.

Notes on customer_id vocabulary:
- Pattern: prefix `CUST-` followed by a 3-digit zero-padded integer (e.g. CUST-001, CUST-009).
- Format is consistent across all sources (seed data and test fixtures).
- CUST-999 is a test-only sentinel — does not exist in the database. Used in test cases verifying 404 behaviour.
- No constraint enforces the CUST-NNN format at the DB level (VARCHAR PRIMARY KEY — any non-null string is valid).

### Risk Factor Inventory (risk_factors field — free-form TEXT[])

risk_factors is a free-form text array — not a controlled vocabulary. Individual factor strings are English phrases; no enum constraint exists. The 21 factor strings observed across 9 seed rows are the full vocabulary of the pre-loaded dataset.

**LOW tier factors (8 unique strings across 3 rows):**

| Row | Factor String |
|---|---|
| CUST-001 | `account in good standing` |
| CUST-001 | `consistent payment history` |
| CUST-001 | `low utilisation rate` |
| CUST-002 | `no missed payments in 24 months` |
| CUST-002 | `single product relationship` |
| CUST-003 | `long tenure` |
| CUST-003 | `stable address history` |
| CUST-003 | `verified income` |

**MEDIUM tier factors (6 unique strings across 3 rows):**

| Row | Factor String |
|---|---|
| CUST-004 | `two late payments in past year` |
| CUST-004 | `moderate utilisation` |
| CUST-005 | `recent address change` |
| CUST-005 | `elevated exposure relative to income` |
| CUST-006 | `disputed transaction on file` |
| CUST-006 | `partial identity verification` |

**HIGH tier factors (7 unique strings across 3 rows):**

| Row | Factor String |
|---|---|
| CUST-007 | `three missed payments` |
| CUST-007 | `account referred to collections` |
| CUST-008 | `fraud alert on file` |
| CUST-008 | `multiple failed authentication attempts` |
| CUST-009 | `default recorded` |
| CUST-009 | `county court judgement` |
| CUST-009 | `high utilisation across all accounts` |

Total: 21 unique factor strings. No string appears in more than one row. No string appears across tiers.

---

## Cardinality Samples

| Relationship | Observed Cardinality | Sample Size | Notes |
|---|---|---|---|
| customer_risk_profiles.risk_tier → value distribution | 3 rows per tier (LOW:3, MEDIUM:3, HIGH:3) | 9 rows total | Perfectly balanced seed set — does not reflect production distribution. |
| customer_risk_profiles.risk_factors → array length | min=2, max=3 per row | 9 rows | LOW: 3,2,3; MEDIUM: 2,2,2; HIGH: 2,2,3. All non-empty. Average 2.3 factors/row. |
| customer_risk_profiles → (single table) | No FK relationships | N/A | Single-table schema — no inter-entity cardinality to report. |

Notes:
- The balanced 3-per-tier distribution is a seed data artifact — annotation should clarify expected production skew (likely HIGH < MEDIUM < LOW or context-dependent).
- All risk_factors arrays in seed data have 2–3 elements. The schema permits empty arrays (NOT NULL does not prevent `{}`). No empty-array row exists in seed data — annotation should clarify whether an empty array is a valid business state.

---

## Null Frequency

| Entity | Field | Nullable (declared) | Null in Seed Data | Notes |
|---|---|---|---|---|
| customer_risk_profiles | customer_id | NO (PRIMARY KEY) | Never null | PK — cannot be null by definition. |
| customer_risk_profiles | risk_tier | NO (NOT NULL) | Never null | NOT NULL constraint. All 9 seed rows have a value. |
| customer_risk_profiles | risk_factors | NO (NOT NULL) | Never null | NOT NULL constraint prevents null array. Empty array `{}` would pass the constraint but does not appear in seed data. This is an annotation candidate — see Naming Pattern Flags. |

No fields are nullable in this schema. No annotation candidate for "nullable but never null in seed". One annotation candidate for the inverse edge case: risk_factors NOT NULL but empty array permitted — whether `{}` is a valid business state is NOT DETERMINABLE from schema or seed data alone.

---

## Naming Pattern Flags

| Concept | Name in Source A | Name in Source B | Notes |
|---|---|---|---|
| utilisation (spelling) | `utilisation` (UK English) — used in 3 factor strings: `low utilisation rate`, `moderate utilisation`, `high utilisation across all accounts` | `utilization` (US English) — NOT observed anywhere in the codebase | UK English spelling used consistently throughout risk_factors seed data. No US variant present. Annotation candidate: confirm intended locale for factor string authoring. |
| risk tier ordering | Implied semantic order LOW < MEDIUM < HIGH | No declared sort order in schema (VARCHAR, no ENUM type) | Schema does not enforce or express ordering. Ordering is implicit from domain knowledge only. Annotation candidate: confirm canonical sort order for presentation and report generation. |
| empty risk_factors | `{}` (empty Postgres array) — schema-valid per NOT NULL | No empty-array rows in seed data | Whether an empty array is a valid business state (e.g. for a newly-onboarded customer with no assessed factors) is not determinable from sources. Annotation candidate. |
| non-existent customer_id | `CUST-999` — test sentinel for 404 | Not a seed row | CUST-999 is a test-fixture-only value. The CUST-NNN pattern is shared with seed data, but the 3-digit space (001–999) is partially reserved by seed (001–009) and partially used as test sentinel. No formal gap list exists. |

---

F02 is complete. discovery/F02_vocabulary_extraction.md must be committed and reviewed before F03 begins.
F03 runs in Claude Desktop (CD) — trigger phrase: "Run Session F03".
