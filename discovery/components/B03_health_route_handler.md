## B03 — health route handler
ID: M-003
Layer: serving
Source file: api/main.py:82–84

---

**Module** — health route handler
**ID** — M-003
**Layer** — serving
**Primary Responsibility** — Return a static JSON health indicator. No database interaction, no authentication, no external dependencies.

**Inputs**
None — no path parameters, no query parameters, no request body, no auth dependency.

**Outputs**
- `{"status": "ok"}` — static Python dict returned directly. FastAPI serialises to JSON. Content-Type: application/json. HTTP 200.
- Exactly one key. No additional fields permitted (INV-13).

**Public Interface**
```
GET /health
```
Registered on M-006 via `@app.get("/health")` at api/main.py:82. No auth dependency.

**Error Behaviour**
No error paths. The response is a static dict literal with no external dependencies. There is no conditional logic, no IO, and no exception-raising code path.

**Known Fragility**
- Simplest module in the system. Only fragility: any code change that adds a field to the return dict (INV-13 violation), calls a database function (INV-09 and INV-13 violation), or adds auth (violates CLAUDE.md scope boundary that explicitly lists GET /health as unauthenticated).
- INV-13 enforcement is structural: the return statement is a single-key dict literal. Adding a key requires an explicit code change — no accidental leakage is possible without modification.

**Change Impact**
- Adding any field to the response body violates INV-13.
- Adding a DB call violates INV-13 (and potentially INV-09 if exceptions are not handled).
- Adding `Depends(verify_api_key)` violates CLAUDE.md scope boundary (GET /health must be unauthenticated).
- Removing the route violates CLAUDE.md scope boundary.

**Callers** — M-006 (FastAPI application bootstrap) dispatches incoming `GET /health` requests
**Calls** — None
**Integration Points Used** — None
