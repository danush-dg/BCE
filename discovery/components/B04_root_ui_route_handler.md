## B04 — root (UI) route handler
ID: M-004
Layer: serving
Source file: api/main.py:87–89 (route handler); api/main.py:7–58 (_HTML constant)

---

**Module** — root (UI) route handler
**ID** — M-004
**Layer** — serving
**Primary Responsibility** — Return an HTMLResponse containing the complete browser UI as the `_HTML` module-level string constant. Serves the customer risk lookup form and its embedded JavaScript without authentication.

**Inputs**
None — no path parameters, no query parameters, no request body, no auth dependency. `_HTML` is a module-level constant; no runtime input affects the response content.

**Outputs**
- `HTMLResponse(content=_HTML)` — HTTP 200, Content-Type: text/html. The response body is the static string defined at api/main.py:7–58 verbatim. 1,607 characters, single request-response cycle.

**Public Interface**
```
GET /
```
Registered on M-006 via `@app.get("/", response_class=HTMLResponse)` at api/main.py:87. No auth dependency.

**_HTML Content Summary** (api/main.py:7–58)

The HTML document contains:
- `<input type="password" id="apiKey">` — API key entry (password field; value is never sent to server by Python code, only by the browser JS fetch call)
- `<input type="text" id="customerId">` — customer ID entry with Enter-key listener triggering `lookup()`
- `<button onclick="lookup()">Look Up</button>`
- `<div id="result">` — output display area

**Embedded JavaScript** (`lookup()` function, lines 31–52 of _HTML):
1. Reads `customerId.value.trim()` and `apiKey.value` from inputs
2. If no ID: sets result text to `'Enter a customer ID.'` and returns
3. Issues `fetch('/customers/' + encodeURIComponent(id), { headers: { 'X-API-Key': key } })`
4. On successful response: parses JSON
   - If `data.detail` is truthy: displays `data.detail` (error message path)
   - Else: renders `data.customer_id`, `data.risk_tier`, and `data.risk_factors` values directly — no transformation, no mapping, no switch statement (INV-11 confirmed)
5. On fetch exception: displays `'Request failed.'`

**INV-11 Enforcement Analysis** — CONFIRMED FROM SOURCE
- `data.customer_id`, `data.risk_tier` rendered as string concatenation without any conditional branching on their values
- `data.risk_factors.map(function(f) { return '  - ' + f; })` — list formatting (bullet prefix) applied uniformly; factor strings themselves rendered verbatim without filtering, reordering, or value-based branching
- No switch statements, no object maps (e.g., `{ LOW: 'Low Risk' }`), no conditional display logic on tier or factor values
- `encodeURIComponent(id)` is URL encoding for the fetch call path — not a transformation of the displayed output

**Error Behaviour**
- Server-side: no error paths. `_HTML` is a static string constant; `HTMLResponse(content=_HTML)` cannot raise unless FastAPI's response machinery itself fails.
- Client-side (within JS): network failure → `'Request failed.'`; API error response → `data.detail` string rendered; empty customer ID → `'Enter a customer ID.'`

**Known Fragility**
- `data.risk_factors.map(...)` — if the API ever returned `risk_factors` as `null` or a non-array (schema regression), JavaScript throws `TypeError: data.risk_factors.map is not a function`. The `catch(e)` block would display `'Request failed.'` — a silent data failure with no diagnostic.
- `_HTML` is a Python string constant — any syntax error introduced into the HTML or JS is a silent runtime failure (malformed HTML served, not a build error). No test in M-010, M-011, or M-012 exercises the UI for JS correctness.
- Coupling: UI change requires api container rebuild and redeploy. No separate deployment path for frontend (R-002).
- `if (data.detail)` — this check is truthy for any non-empty `data.detail`. If the API were ever changed to include a `detail` field in a 200 response, the JS would render the error path instead of the data path. Not currently possible given INV-05 enforcement, but a latent coupling.

**Change Impact**
- Any JS modification that adds a switch statement or value map on `risk_tier` or that filters `risk_factors` violates INV-11.
- Adding a `<script src="...">` or CDN link violates INV-10 (external network call from UI) and CLAUDE.md fixed stack (no CDN).
- Adding `Depends(verify_api_key)` violates CLAUDE.md scope boundary (GET / must be unauthenticated).

**Callers** — M-006 (FastAPI application bootstrap) dispatches incoming `GET /` requests; HTTP client (browser)
**Calls** — None (JS fetch calls to M-005 are browser-side runtime calls, not server-side module dependencies)
**Integration Points Used** — None
