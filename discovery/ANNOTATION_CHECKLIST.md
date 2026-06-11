STAGE-3-COMPLETE: CROSS-ARTIFACT REVIEW — 2026-06-11 — BCE Adapter Pipeline Stage 3 (CD)

# ANNOTATION_CHECKLIST.md — Customer Risk API
Produced by: BCE Adapter Pipeline Stage 3 (CD) — 2026-06-11
Sources: All six BCE artifacts in discovery/ + discovery/DOMAIN_MODEL.json

Schema validation: PASS
Checks completed: CHECK 0 through CHECK 6
P1 items must be SIGNED-OFF or RESOLVED by the engineer before Stage 3 completes.

---

## P1 Items

---

## P1-S3-001 · IC-9 enforcement gap — cursor exceptions bypass static literal (MODULE_CONTRACTS + INVARIANT_CATALOGUE)

**Severity:** P1
**Type:** CONTRADICTION
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** INVARIANT_CATALOGUE + MODULE_CONTRACTS
**Section:** IC-9 Enforcement point; M-005 Error Behaviour

**Observation:** INVARIANT_CATALOGUE IC-9 (Internal Error Isolation) states enforcement point is M-002 `get_db_connection` and cites `except Exception:` at main.py:76 as the enforcement mechanism. However, MODULE_CONTRACTS M-005 Error Behaviour (line 184) explicitly states: "Exception from `cur.execute()`/`cur.fetchone()` after successful connect propagates to FastAPI default handler — 500 with framework-generated body (not the controlled static literal)." The FastAPI default 500 body is `{"detail": "Internal Server Error"}` — capitalised, different from the system's permitted literal `"Internal server error"` (lowercase s). If psycopg2 raises after the connection is established (e.g., query execution error, cursor operation failure), the 500 response body is not the controlled static literal. IC-9 as written implies the static literal covers all 500 paths — this is false.

**Risk for planning:** Any enhancement that introduces conditions where `cur.execute()` or `cur.fetchone()` could raise (e.g., schema changes, query modifications, timeout injection) would produce a non-compliant 500 response, violating IC-9 without any test catching it. M-012 TestInv09InternalErrorIsolation tests only the connection-failure path (stops the db container) — it does not test the cursor-exception path.

**Recommended action:** Engineer must determine the correct scoping of IC-9: (a) If IC-9 is intended to cover only connection failures (M-002 path), update IC-9 statement and enforcement point to reflect this boundary explicitly. This is an honest narrowing. (b) If IC-9 is intended to cover all 500 paths including cursor exceptions, add a `try/except Exception` inside the `try` block in M-005 around `cur.execute()` / `cur.fetchone()`, raising `HTTPException(500, "Internal server error")` — and add a test that verifies this. Option (a) is the lower-effort resolution; option (b) is the more complete invariant.

**Engineer action required:** Choose (a) or (b). Update IC-9 enforcement point field accordingly. If option (b): update M-005 and add test. Sign off with rationale.

**STATUS:** RESOLVED — 2026-06-11
Option (a) chosen. IC-9 statement narrowed to explicitly scope the controlled-literal guarantee to connection-failure, auth-failure, and not-found paths. Cursor-level exceptions in M-005 acknowledged as falling to FastAPI's default handler — non-revealing, capitalisation difference accepted as an honest boundary. ENGINEER NOTE applied to IC-9 in INVARIANT_CATALOGUE.md. Option (b) deferred.

---

## P1-S3-002 · STAGE-2-DIVERGENCE unresolved — Postgres port 5432 exposed on host (MODULE_CONTRACTS M-008)

**Severity:** P1
**Type:** CONTRADICTION
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS + RISK_REGISTER
**Section:** M-008 Known Fragility; R-006

**Observation:** MODULE_CONTRACTS M-008 contains a `STAGE-2-DIVERGENCE` flag (2026-06-11): Stage 1 described container isolation by noting "no external networks defined." Code shows `ports: 5432:5432` exposing Postgres on host port 5432. This divergence is recorded in R-006 (P2, Open) but has not been resolved or signed off — it remains a live STAGE-2-DIVERGENCE marker in MODULE_CONTRACTS. Per the methodology: every STAGE-2-DIVERGENCE must be resolved or have an engineer sign-off before Stage 3 completes.

**Risk for planning:** Without sign-off, this divergence is an unresolved design-vs-reality gap. For any deployment beyond a solo developer workstation, `ports: 5432:5432` is a security boundary violation — callers with Postgres credentials bypass all application-layer invariants.

**Recommended action:** Engineer signs off on this divergence with one of: (a) "Accepted for dev scope — will remove `ports: 5432:5432` before any shared or production deployment" — update R-006 status accordingly. (b) Remove `ports: 5432:5432` from docker-compose.yml now (one-line change, no application impact). Sign off in MODULE_CONTRACTS M-008 Known Fragility with an ENGINEER NOTE.

**Engineer action required:** Sign off on the divergence with explicit disposition. Add ENGINEER NOTE to M-008. Update R-006 status.

**STATUS:** RESOLVED — 2026-06-11
Accepted for development scope. ENGINEER NOTE applied to M-008 Known Fragility in MODULE_CONTRACTS.md: `ports: 5432:5432` accepted for single-developer internal ops deployment; must be removed before any shared-host or production deployment. R-006 remains Open (P2) as the action item tracking this. Resolves the Stage-2-Divergence sign-off requirement.

---

## P2 Items

---

## P2-S3-001 · Positional column access in M-005 — silent transposition risk not in RISK_REGISTER (MODULE_CONTRACTS)

**Severity:** P2
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS + RISK_REGISTER
**Section:** M-005 Known Fragility

**Observation:** MODULE_CONTRACTS M-005 Known Fragility identifies that `row[0]`, `row[1]`, `row[2]` positional access is bound to SELECT column order. A column reorder in the SELECT query or schema silently transposes `customer_id`, `risk_tier`, and `risk_factors` in the 200 response — violating IC-5 (response shape) and IC-6 (risk tier values) without raising an exception. This fragility has no corresponding RISK_REGISTER entry and no test that would catch it.

**Risk for planning:** Any enhancement touching the SELECT query in M-005 or the schema in M-007 must be reviewed against this fragility. If a developer changes `SELECT customer_id, risk_tier, risk_factors` to `SELECT risk_tier, customer_id, risk_factors`, M-012 TestInv05ResponseShape checks field presence but not field-value correspondence — the transposition would pass the test suite.

**Recommended action:** Add to RISK_REGISTER as a P2 or P3 risk. Consider using named column access (`cur.description` + dict construction, or `psycopg2.extras.RealDictCursor`) instead of positional indexing — eliminates the fragility entirely and is a low-risk refactor. At minimum, add a comment in M-005 source anchoring the positional indices to the column names.

**Engineer action required:** Decide: add risk entry (defensive) or refactor to named access (preferred). Sign off.

**STATUS:** OPEN

---

## P2-S3-002 · No query timeout in M-005 — slow DB query holds uvicorn worker (MODULE_CONTRACTS)

**Severity:** P2
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS + RISK_REGISTER
**Section:** M-005 Known Fragility

**Observation:** MODULE_CONTRACTS M-005 Known Fragility identifies no query timeout on the psycopg2 cursor. A slow or hanging DB query holds the single uvicorn worker thread indefinitely (M-009 runs single worker). Under a stuck query, the API becomes unresponsive to all subsequent requests. This is not captured in RISK_REGISTER.

**Risk for planning:** Any schema change or query modification that could cause table scans or lock contention creates a denial-of-service risk for the API. Given the single-worker architecture (M-009 single uvicorn worker), one stuck query = total API unavailability.

**Recommended action:** Add to RISK_REGISTER. Mitigation options: set `options='-c statement_timeout=5000'` in psycopg2.connect() (5s timeout — any query longer than this raises OperationalError, caught by M-002's `except Exception:`). Low-effort, high-value mitigation for a read-only query that should return in milliseconds.

**Engineer action required:** Decide whether to add a statement timeout or accept and document the risk. Sign off.

**STATUS:** OPEN

---

## P2-S3-003 · Container runs as root — no USER instruction in Dockerfile (MODULE_CONTRACTS)

**Severity:** P2
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS + RISK_REGISTER
**Section:** M-009 Known Fragility

**Observation:** MODULE_CONTRACTS M-009 Known Fragility identifies that the api container runs as root (no `USER` instruction in Dockerfile). Not captured in RISK_REGISTER. A container escape or RCE vulnerability in FastAPI, uvicorn, or psycopg2 runs with root privileges inside the container.

**Risk for planning:** Acceptable for a developer workstation tool. P2 for any shared-host or production deployment — same deployment context as R-006.

**Recommended action:** Add to RISK_REGISTER at P3 (dev scope) or P2 (if production deployment is anticipated). Mitigation: add `RUN useradd -m appuser && USER appuser` to Dockerfile before CMD — one-line change, no application impact, dramatically reduces container breakout blast radius.

**Engineer action required:** Decide on severity relative to deployment scope. Add to RISK_REGISTER. Sign off.

**STATUS:** OPEN

---

## P2-S3-004 · IC-10, IC-11, IC-14 have no automated test — coverage gap not linked to specific ANNOTATION_CHECKLIST item (INVARIANT_CATALOGUE)

**Severity:** P2
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** INVARIANT_CATALOGUE + RISK_REGISTER
**Section:** Automated Coverage Summary; R-007

**Observation:** INVARIANT_CATALOGUE coverage gap (IC-10, IC-11, IC-14 — manual only) is captured in R-007 in the RISK_REGISTER. R-007 provides recommended actions. This item formalises the gap as an annotation checklist entry to ensure it is explicitly signed off by the engineer and not quietly carried forward.

**Risk for planning:** Regressions against IC-10 (external imports), IC-11 (JS transformation), and IC-14 (connection closure) are not caught by any automated test. Changes to M-004, M-005, M-006, or api/requirements.txt that violate these invariants pass CI silently.

**Recommended action:** Implement at least the IC-10 grep check (lowest effort — a one-line CI command) and the IC-14 connection-count test (highest value — catches silent outage mode). See R-007 recommended action for full specification.

**Engineer action required:** Prioritise and schedule automated test additions. Sign off with implementation plan or explicit acceptance of manual-only coverage.

**STATUS:** OPEN

---

## P2-S3-005 · API key entropy and rotation policy not documented — OQ-04 partially unresolved (INTAKE_SUMMARY)

**Severity:** P2
**Type:** OPEN_QUESTION
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** INTAKE_SUMMARY + INTEGRATION_CONTRACTS
**Section:** Open Questions — OQ-04

**Observation:** INTAKE_SUMMARY OQ-04 (API key format and minimum entropy) is marked "Partially resolved — remains an open operational consideration for production hardening." No BCE artifact captures the API key rotation policy, minimum length, or entropy requirements. The application uses `os.environ.get("API_KEY")` — any non-empty string is a valid key. A one-character key satisfies the auth contract structurally.

**Risk for planning:** For any consumer of this system's SIL (System Intelligence Layer) as a planning input, the API key security posture is a gap. Any enhancement that exposes the API to a wider audience requires a documented key policy.

**Recommended action:** Document in INTEGRATION_CONTRACTS.md or RISK_REGISTER.md: minimum key length, rotation frequency, and key distribution process. Add as ENGINEER NOTE once confirmed. Not a code change — an operational documentation gap.

**Engineer action required:** Supply minimum entropy requirements and rotation policy as ENGINEER NOTE in INTEGRATION_CONTRACTS IP-001 auth mechanism field.

**STATUS:** OPEN

---

## P3 Items

---

## P3-S3-001 · M-004 JavaScript `data.detail` coupling to API response shape (MODULE_CONTRACTS)

**Severity:** P3
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS
**Section:** M-004 Known Fragility

**Observation:** MODULE_CONTRACTS M-004 Known Fragility identifies: "`if (data.detail)` check couples to API response shape — any future 200 response with a `detail` field would render the error path." Not in RISK_REGISTER. Within current PBVI scope (three fixed routes, no new endpoints), this coupling cannot trigger because the only 200 response is the customer record which has no `detail` field. Risk activates on any future route addition returning a JSON object with a `detail` key.

**Risk for planning:** Low within current scope. P3 annotation for future enhancement planning — any enhancement adding a 200 response shape must review this coupling.

**Recommended action:** Add comment in `_HTML` JavaScript anchoring the `data.detail` check to FastAPI HTTPException response shape. Consider changing to explicit status-code check (`if (res.status !== 200)`) which is more robust. No urgency within current scope.

**Engineer action required:** Acknowledge. Annotate if desired.

**STATUS:** OPEN

---

## P3-S3-002 · `_HTML` JavaScript syntax errors are silent at build time (MODULE_CONTRACTS)

**Severity:** P3
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS
**Section:** M-004 Known Fragility

**Observation:** `_HTML` is a Python string constant. A JavaScript syntax error inside it is not caught at Python parse time, import time, or test time — it is only discovered when a browser attempts to parse and execute the script. M-012 does not include a browser-execution test. Within current PBVI scope, the JavaScript is confirmed working (VERIFICATION_RECORD.md). Risk activates on any future modification to `_HTML`.

**Risk for planning:** Low within current scope. P3 annotation for future enhancement awareness.

**Recommended action:** Add a sanity-check test that fetches `GET /` and verifies the response body contains the expected function name (`lookup`) as a smoke-test proxy for "JS is present and roughly intact." Not a substitute for browser execution testing but catches gross truncation errors.

**Engineer action required:** Acknowledge.

**STATUS:** OPEN

---

## P3-S3-003 · TestInv09 destructive lifecycle — db container left stopped on teardown failure (MODULE_CONTRACTS)

**Severity:** P3
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** MODULE_CONTRACTS
**Section:** M-012 Known Fragility

**Observation:** MODULE_CONTRACTS M-012 Known Fragility: TestInv09 calls `docker compose stop db` / `docker compose start db`. If `tearDownClass` fails, the db container remains stopped until manual recovery. The subsequent test suites (M-010, M-011) would all fail with connection errors, masking the real results. Not captured in RISK_REGISTER.

**Risk for planning:** Low — affects only local test runs, not production. The 34/34 pass in VERIFICATION_RECORD.md confirms it worked in verified condition. P3 for test reliability.

**Recommended action:** Add a `try/finally` guard in `tearDownClass` to ensure `docker compose start db` runs even if intermediate assertions fail. Or run M-012 last in the test execution order.

**Engineer action required:** Acknowledge.

**STATUS:** OPEN

---

## P3-S3-004 · DOMAIN_MODEL.json annotation candidates — business_definition, lifecycle_summary, domain on E-001 (DOMAIN_MODEL)

**Severity:** P3
**Type:** NOT_DETERMINABLE
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** DOMAIN_MODEL.json
**Section:** Entity E-001; Attributes A-001/A-002/A-003; StatusValues SVV-001/002/003

**Observation:** Session F03 produced 11 NOT_DETERMINABLE fields across E-001 (business_definition, lifecycle_summary, domain), A-001/A-002/A-003 (business_meaning), A-003 (null_semantic — empty array edge case), and SVV-001/002/003 (business_meaning, is_terminal, transition_trigger). These require engineer annotation. Additionally, three F02 naming pattern flags are unresolved: risk_tier ordering (implied LOW < MEDIUM < HIGH, not declared), UK English spelling in risk_factors, and empty risk_factors array validity.

**Risk for planning:** These NOT_DETERMINABLE fields reduce the precision of module contract extraction in future BCE sessions and limit the utility of DOMAIN_MODEL.json as a planning artifact. Not blocking for current system operation — the API reads and returns data without semantic interpretation.

**Recommended action:** Run the Domain Model annotation pass before Sessions B and C of any future enhancement. At minimum resolve: (1) E-001 domain — which business domain owns this entity; (2) A-003 null_semantic — is `{}` a valid business state for risk_factors; (3) SVV-001/002/003 is_terminal — are any tier values terminal states.

**Engineer action required:** Annotation pass on DOMAIN_MODEL.json. Use ENGINEER NOTE format. Produce HAR per resolved item per the HAR specification in Section 8.

**STATUS:** OPEN

---

## P3-S3-005 · psycopg2 TEXT[] → Python list mapping assumption not explicitly tested (INTEGRATION_CONTRACTS)

**Severity:** P3
**Type:** OPEN_QUESTION
**Source:** STAGE3_REVIEW
**Surfaced by:** CD
**Artifact:** INTEGRATION_CONTRACTS
**Section:** IP-001 — What the application assumes it will receive

**Observation:** INTEGRATION_CONTRACTS flags (as a gap) that psycopg2 mapping of `TEXT[]` to a Python list is an assumption — not verified by an explicit type assertion in any test. IC-5 passing provides implicit coverage only. Standard psycopg2 behaviour supports this mapping, but no test asserts `isinstance(row[2], list)` before serialisation.

**Risk for planning:** Very low — standard psycopg2 behaviour is stable and well-documented. P3 informational.

**Recommended action:** Add a single assertion in M-012 or M-010 confirming `type(data["risk_factors"]) == list`. Five minutes of effort; eliminates the assumption.

**Engineer action required:** Acknowledge or implement assertion.

**STATUS:** OPEN

---

## Resolution Log

| Item ID | Resolution type | Resolved by | Date | Evidence |
|---|---|---|---|---|
| P1-S3-001 | RESOLVED-ANNOTATION | Engineer | 2026-06-11 | IC-9 narrowed in INVARIANT_CATALOGUE.md; ENGINEER NOTE applied. Option (a) chosen — cursor-exception path accepted as honest boundary. |
| P1-S3-002 | RESOLVED-ANNOTATION | Engineer | 2026-06-11 | ENGINEER NOTE applied to M-008 Known Fragility in MODULE_CONTRACTS.md. Dev-scope acceptance stated; R-006 (P2) tracks production hardening action. |

---

## Cross-Artifact Consistency Check

**Last run:** 2026-06-11 **By:** CD

| Check | Status | Notes |
|---|---|---|
| All invariants in INVARIANT_CATALOGUE.md match INVARIANTS.md | NOT DETERMINABLE | docs/invariant.md not uploaded to this session. INVARIANT_CATALOGUE header states all 13 Stage 1 IC records sourced from docs/invariant.md and confirmed in Stage 2 with no divergences. Indirect confidence HIGH — direct check requires docs/invariant.md to be present. Engineer should verify once. |
| All module names in MODULE_CONTRACTS.md match TOPOLOGY.md | PASS | All 12 module names (M-001 through M-012) match exactly across both artifacts. Layer assignments consistent. |
| All external systems in INTEGRATION_CONTRACTS.md match TOPOLOGY.md A03 | PASS | One external system: IP-001 Postgres 15. Name, role, connection mechanism, and called-by fields consistent across both artifacts. |
| All risks in RISK_REGISTER.md reference correct source artifacts | PASS | All 8 risk entries (R-001 through R-008) reference M-NNN and IC-N. Affected module and invariant fields resolved in Session E. Source artifact fields reference docs/architecture.md, MODULE_CONTRACTS.md, INTEGRATION_CONTRACTS.md, INVARIANT_CATALOGUE.md — all valid. |
| INTAKE_SUMMARY.md open questions accounted for in artifacts | PASS (partial) | OQ-01/02/03 fully resolved (confirmed in INTAKE_SUMMARY). OQ-04 (API key entropy) partially resolved — captured as P2-S3-005. |
| Entity names in DOMAIN_MODEL.json consistent with domain terminology in INVARIANT_CATALOGUE.md and MODULE_CONTRACTS.md | PASS | `CustomerRiskProfile` (E-001) consistent with `customer_risk_profiles` table references throughout INVARIANT_CATALOGUE and MODULE_CONTRACTS. Attribute names (customer_id, risk_tier, risk_factors) match exactly across all artifacts. StatusVocabulary values (LOW, MEDIUM, HIGH) match IC-6 and M-007 CHECK constraint. |

---

## Stage 3 Completeness Summary — 2026-06-11
Produced by: BCE Adapter Pipeline Stage 3 (CD)

| Severity | Count | Disposition |
|---|---|---|
| P1 | 2 | Must be SIGNED-OFF or RESOLVED before Stage 3 completes |
| P2 | 5 | Resolve before first enhancement |
| P3 | 5 | Informational backlog |
| CON | 0 | — |
| **Total** | **12** | |

**Consistency check: PASS (with 1 NOT DETERMINABLE** — INVARIANTS.md not available for direct cross-check; engineer verification recommended)

**P1 items requiring engineer action:**

- **P1-S3-001** — IC-9 enforcement gap: cursor exceptions in M-005 bypass the controlled static literal. Engineer must choose resolution path (a) narrow the IC-9 statement or (b) add exception handling in M-005 and a test. This is a genuine correctness gap in the invariant as written.
- **P1-S3-002** — M-008 STAGE-2-DIVERGENCE (Postgres port 5432 on host) not signed off. Engineer must add ENGINEER NOTE to M-008 with explicit disposition.

**Stage 3 is complete when all P1 items are SIGNED-OFF or RESOLVED by the engineer.**

---

## Stage 3 Close-Out Step — Graph Construction

After all P1 items are SIGNED-OFF or RESOLVED: run the graph construction prompt in CC.
Trigger phrase: **"Build system graph"**
Output: `discovery/SYSTEM_GRAPH.json`
DOMAIN_MODEL.json is present and non-sparse — cross-graph edges (OWNS_ENTITY, READS_ENTITY, ENFORCES_ENTITY_INVARIANT, ENFORCES_ATTRIBUTE_INVARIANT) will be included automatically.
This step is mandatory. Stage 3 is not complete until SYSTEM_GRAPH.json is committed to discovery/.
