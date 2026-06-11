## C01 — FastAPI application bootstrap
ID: M-006
Layer: infra
Source file: api/main.py:60

---

**Module** — FastAPI application bootstrap
**ID** — M-006
**Layer** — infra
**Primary Responsibility** — Instantiate the FastAPI ASGI application object at module load time. All route registrations, dependency injection wiring, and framework dispatch happen through this object.

**Inputs**
- `FastAPI()` constructor called with no arguments — default settings. No custom exception handlers, no middleware, no CORS configuration, no dependency overrides.

**Outputs**
- `app` — a FastAPI ASGI application instance, module-level global. Used as:
  - Target of `@app.get(...)` decorators at lines 82, 87, 92 — registers M-003, M-004, M-005
  - Argument to uvicorn: `CMD ["uvicorn", "main:app", ...]` in M-009

**Public Interface**
- `app` is referenced by uvicorn at startup via `main:app` (module:attribute notation).
- `@app.get("/health")` — registers M-003
- `@app.get("/")` — registers M-004 with `response_class=HTMLResponse`
- `@app.get("/customers/{customer_id}")` — registers M-005 with `Depends(verify_api_key)`

**Error Behaviour**
- `FastAPI()` instantiation: STARTUP-FATAL if FastAPI package is not installed (ImportError at module load, before any request can be served).
- Route registration via decorators: STARTUP-FATAL if any decorator raises (extremely unlikely with no custom configuration).
- No custom exception handlers registered — unhandled exceptions in route handlers produce FastAPI's default 500 response with framework-generated content, not the system's controlled static literals (R-001 path).

**Known Fragility**
- Default FastAPI exception handling: if an exception escapes a route handler's explicit `except` clause, FastAPI generates a 500 with its own default body. This body may contain information beyond the system's permitted error literals (INV-09). For the current three routes, all exception paths are explicitly handled — but this is a per-route responsibility, not enforced globally here.
- No middleware: CORS, rate limiting, request logging, and request timeout are absent. Not required by current scope, but this is the place they would be added.
- No global dependency on `verify_api_key`: auth is wired per-route. A future route added without `Depends(verify_api_key)` would be unauthenticated (R-005). A global dependency here would prevent this, but would require excluding `/health` and `/` from the auth gate.

**Change Impact**
- Adding middleware or exception handlers affects all routes globally.
- Adding a global dependency modifies the auth behaviour of every route.
- This module is referenced by name in M-009 (`main:app`) — renaming `app` or the module would break the container startup command.

**Callers** — M-009 (Dockerfile CMD references `main:app`); uvicorn runtime
**Calls** — M-003, M-004, M-005 (framework dispatch on incoming requests)
**Integration Points Used** — None
