# ID_REGISTRY.md — Customer Risk API
Produced by: BCE Stage 2 Session A (CC)
Date: 2026-06-11

This file is the canonical permanent ID registry for the Customer Risk API BCE extraction.
IDs are never reassigned or reused. Sessions B, C, D, and E must reference M-NNN and IP-NNN identifiers from this registry.

---

## Module IDs (M-NNN)

| ID | Module Name | Source File | Layer | Assigned |
|---|---|---|---|---|
| M-001 | verify_api_key | api/main.py:63–66 | serving | 2026-06-11 Session A |
| M-002 | get_db_connection | api/main.py:69–79 | serving | 2026-06-11 Session A |
| M-003 | health route handler | api/main.py:82–84 | serving | 2026-06-11 Session A |
| M-004 | root (UI) route handler | api/main.py:87–89 | serving | 2026-06-11 Session A |
| M-005 | get_customer route handler | api/main.py:92–106 | serving | 2026-06-11 Session A |
| M-006 | FastAPI application bootstrap | api/main.py:60 | infra | 2026-06-11 Session A |
| M-007 | database initialisation | db/init.sql | infra | 2026-06-11 Session A |
| M-008 | container orchestration | docker-compose.yml | infra | 2026-06-11 Session A |
| M-009 | container image definition | api/Dockerfile | infra | 2026-06-11 Session A |
| M-010 | error state test suite | api/test_errors.py | infra | 2026-06-11 Session A |
| M-011 | SQL injection test suite | api/test_injection.py | infra | 2026-06-11 Session A |
| M-012 | full invariant verification suite | api/test_invariants.py | infra | 2026-06-11 Session A |

Next M-NNN to assign: M-013

---

## Integration Point IDs (IP-NNN)

| ID | System Name | Type | Location | Called By | Assigned |
|---|---|---|---|---|---|
| IP-001 | Postgres 15 | database | db container (Compose internal network) | M-002, M-005 (via M-002) | 2026-06-11 Session A |

Next IP-NNN to assign: IP-002

---

## Stage 1 Skeleton ID Cross-reference

Stage 1 draft MODULE_CONTRACTS.md used S1-NN placeholder IDs. This table maps those to permanent M-NNN IDs for Session B/C contract authoring.

| Stage 1 Skeleton ID | Permanent Module ID | Module Name |
|---|---|---|
| S1-01 | M-001 | verify_api_key |
| S1-02 | M-002 | get_db_connection |
| S1-03 | M-005 | get_customer route handler |
| S1-04 | M-004 | root (UI) route handler |
| S1-05 | M-003 | health route handler |
| S1-06 | M-007 | database initialisation |
| S1-07 | M-008 | container orchestration |

Additional modules assigned in Stage 2 (no Stage 1 skeleton): M-006, M-009, M-010, M-011, M-012.

---

## Registry Notes

- The Stage 1 skeletons did not include the FastAPI application bootstrap (M-006), the Dockerfile (M-009), or the three test suites (M-010, M-011, M-012) as distinct modules. These were identified from source code in Session A0/A.
- No external systems beyond Postgres 15 were identified. No further IP-NNN assignments expected unless a new integration point is introduced.
- This registry is append-only. Do not remove rows or reuse IDs. If a module is removed from the codebase, annotate with `[REMOVED: date]` rather than deleting the row.
