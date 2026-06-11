## B01 — verify_api_key
ID: M-001
Layer: serving
Source file: api/main.py:63–66

---

**Module** — verify_api_key
**ID** — M-001
**Layer** — serving
**Primary Responsibility** — Validate the `X-API-Key` request header against the `API_KEY` environment variable. Raise HTTPException(401, "Unauthorized") for any invalid, missing, or empty key before any route body executes.

**Inputs**
- `api_key: str` — value extracted from `X-API-Key` request header via `Header(None, alias="X-API-Key")`. FastAPI injects `None` if the header is absent; empty string if the header is present but blank.
- `API_KEY` environment variable — read via `os.environ.get("API_KEY")` at call time (not at module load time). Returns `None` if the variable is unset.

**Outputs**
- No return value on success — function returns `None` implicitly. FastAPI dependency resolution continues to the route body.
- Raises `HTTPException(status_code=401, detail="Unauthorized")` on any failure case. Detail is a static string literal — not an f-string, not derived from the submitted key value or the failure mode.

**Public Interface**
```python
async def verify_api_key(api_key: str = Header(None, alias="X-API-Key")) -> None
```
Declared as `Depends(verify_api_key)` on M-005 (get_customer). Not attached to M-003 (GET /health) or M-004 (GET /).

**Error Behaviour**
All failure paths emit a single raise statement — a structural guarantee that INV-12 cannot be violated by incomplete code changes.

| Input condition | Condition triggered | Result |
|---|---|---|
| `X-API-Key` header absent | `not api_key` is True (api_key is None) | HTTPException(401, "Unauthorized") |
| `X-API-Key: ` (empty string) | `not api_key` is True (empty string is falsy) | HTTPException(401, "Unauthorized") |
| `X-API-Key: wrong-value` | `api_key != valid_key` is True | HTTPException(401, "Unauthorized") |
| `API_KEY` env var unset (valid_key = None) | `api_key != None` is True for any non-None api_key | HTTPException(401, "Unauthorized") |
| `X-API-Key: correct-value` | both conditions False | Returns None — route body executes |

**Known Fragility**
- `os.environ.get("API_KEY")` is intentionally different from `os.environ["API_KEY"]`. The `.get()` variant returns `None` if the env var is unset. `None != anything` is always True, so all requests fail auth with 401 rather than raising an unhandled KeyError. This is fail-safe by design — a misconfigured deployment produces 401s, not 500s.
- Per-route dependency injection: any new route added without `Depends(verify_api_key)` creates an unauthenticated endpoint. CLAUDE.md scope boundary prohibits new routes within PBVI scope, but this is a fragility for any future extension (R-005).
- The single `if` condition with single `raise` enforces INV-12 structurally. If this were refactored into separate conditionals with separate raises, INV-12 would require explicit re-verification.

**Change Impact**
- Changing the `detail` string violates `execution_plan.md` permitted error literals and INV-12.
- Splitting the condition into separate raises per failure mode (missing vs. wrong key) violates INV-12.
- Adding logging of `api_key` or `valid_key` violates INV-08.
- Changing `os.environ.get` to `os.environ["API_KEY"]` changes failure mode on unset env var from 401 to 500.

**Callers** — M-005 (get_customer route handler) via `Depends(verify_api_key)`
**Calls** — None (reads environment variable only; no module calls)
**Integration Points Used** — None
